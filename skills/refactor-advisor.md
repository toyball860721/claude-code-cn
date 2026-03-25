# 🔧 重构建议 (Refactor Advisor)

## 描述

分析代码结构，识别代码坏味道（Code Smells），给出具体的重构方案。支持提取函数、消除重复、简化条件、应用设计模式等常见重构手法，输出重构前后的对比代码和步骤说明。

## 使用场景

- 代码变得难以维护时的结构优化
- 技术债清理前的评估
- 大函数/大类的拆分指导
- 引入设计模式替换硬编码
- Code Review 中发现的结构问题修复

## Prompt 模板

```
请分析以下代码的结构问题并给出重构建议。

分析维度：
1. **代码坏味道识别**（过长函数、重复代码、过深嵌套、上帝类、特性依恋等）
2. **重构手法推荐**（提取函数、引入参数对象、策略模式、组合替代继承等）
3. **优先级排序**（按影响范围和实施难度排序）
4. **重构步骤**（逐步操作，每步保持代码可运行）
5. **重构后代码**（完整可用的最终代码）

约束：
- 重构不改变外部行为
- 每个步骤都应可独立提交
- 标注哪些步骤需要同步修改测试

代码如下：
```{language}
{code}
```
```

## 示例输入

```python
def process_order(order):
    # 验证
    if order['items'] is None or len(order['items']) == 0:
        return {"status": "error", "msg": "没有商品"}
    if order['customer'] is None:
        return {"status": "error", "msg": "没有客户信息"}
    if order['customer']['email'] is None or '@' not in order['customer']['email']:
        return {"status": "error", "msg": "邮箱无效"}

    # 计算
    total = 0
    for item in order['items']:
        if item['type'] == 'physical':
            total += item['price'] * item['qty']
            if item['price'] * item['qty'] > 500:
                total -= item['price'] * item['qty'] * 0.1
        elif item['type'] == 'digital':
            total += item['price'] * item['qty']
            if item['price'] * item['qty'] > 200:
                total -= item['price'] * item['qty'] * 0.15
        elif item['type'] == 'subscription':
            total += item['price'] * item['months']

    # 发送
    if order['customer']['vip']:
        total *= 0.9
    tax = total * 0.13
    send_email(order['customer']['email'], f"订单总额: {total + tax}")
    save_to_db(order, total, tax)
    return {"status": "ok", "total": total, "tax": tax}
```

## 示例输出

```markdown
# 重构报告

## 识别的代码坏味道

| # | 坏味道 | 位置 | 严重度 |
|---|--------|------|--------|
| 1 | 过长函数 | 整个函数（35行，3个职责）| 🔴 高 |
| 2 | 重复代码 | 价格计算中的 `price * qty` | 🟡 中 |
| 3 | 魔法数字 | 0.1, 0.15, 0.9, 0.13 | 🟡 中 |
| 4 | 基本类型偏执 | 全程使用 dict 而非对象 | 🟡 中 |
| 5 | 过长条件链 | 验证部分的三层 if | 🔵 低 |

## 重构步骤

### 步骤 1：提取验证函数
```python
def validate_order(order: dict) -> str | None:
    """验证订单，返回错误信息或 None"""
    if not order.get('items'):
        return "没有商品"
    if not order.get('customer'):
        return "没有客户信息"
    email = order['customer'].get('email', '')
    if not email or '@' not in email:
        return "邮箱无效"
    return None
```

### 步骤 2：提取单项价格计算 + 消除魔法数字
```python
DISCOUNT_RULES = {
    'physical':     {'threshold': 500, 'rate': 0.10},
    'digital':      {'threshold': 200, 'rate': 0.15},
}
VIP_DISCOUNT = 0.10
TAX_RATE = 0.13

def calc_item_subtotal(item: dict) -> float:
    if item['type'] == 'subscription':
        return item['price'] * item['months']
    subtotal = item['price'] * item['qty']
    rule = DISCOUNT_RULES.get(item['type'])
    if rule and subtotal > rule['threshold']:
        subtotal *= (1 - rule['rate'])
    return subtotal
```

### 步骤 3：组合为清晰的主函数
```python
def process_order(order: dict) -> dict:
    error = validate_order(order)
    if error:
        return {"status": "error", "msg": error}

    subtotal = sum(calc_item_subtotal(item) for item in order['items'])
    if order['customer'].get('vip'):
        subtotal *= (1 - VIP_DISCOUNT)
    tax = subtotal * TAX_RATE

    send_email(order['customer']['email'], f"订单总额: {subtotal + tax:.2f}")
    save_to_db(order, subtotal, tax)
    return {"status": "ok", "total": subtotal, "tax": tax}
```

## 重构收益
- 主函数从 35 行缩减到 12 行
- 每个函数单一职责，可独立测试
- 折扣规则集中管理，新增商品类型只需改配置
- 消除了所有魔法数字
```
