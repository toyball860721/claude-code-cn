# 🧪 自动写单元测试 (Test Writer)

## 描述

根据源代码自动生成全面的单元测试，支持 Python(pytest)、TypeScript(Jest/Vitest)、Go(testing)、Java(JUnit) 等主流框架。覆盖正常路径、边界条件、异常处理，并生成测试描述的中文注释。

## 使用场景

- 为已有代码补充单元测试
- 新功能开发时先写测试（TDD 辅助）
- 提高项目测试覆盖率
- 重构前建立安全网
- CI/CD 流水线中的测试补全

## Prompt 模板

```
请为以下代码生成完整的单元测试。

要求：
1. **测试框架**：{framework}（如 pytest / Jest / Vitest / JUnit）
2. **覆盖范围**：
   - 正常输入（至少3个用例）
   - 边界条件（空值、零值、极大值、空列表等）
   - 异常/错误路径（无效输入、权限不足、网络超时等）
   - 如有数据库/外部服务，使用 mock
3. **命名规范**：测试函数名用英文，注释用中文描述测试意图
4. **输出**：可直接运行的测试文件

代码如下：
```{language}
{code}
```

依赖上下文（如有）：{context}
```

## 示例输入

```python
class Calculator:
    def divide(self, a: float, b: float) -> float:
        if b == 0:
            raise ValueError("除数不能为零")
        return a / b

    def batch_average(self, numbers: list[float]) -> float:
        if not numbers:
            raise ValueError("列表不能为空")
        return sum(numbers) / len(numbers)
```

## 示例输出

```python
import pytest
from calculator import Calculator


class TestCalculatorDivide:
    """测试 Calculator.divide 方法"""

    def setup_method(self):
        self.calc = Calculator()

    # ---- 正常路径 ----

    def test_divide_positive_numbers(self):
        """正整数相除"""
        assert self.calc.divide(10, 2) == 5.0

    def test_divide_negative_numbers(self):
        """负数相除"""
        assert self.calc.divide(-10, -2) == 5.0

    def test_divide_returns_float(self):
        """结果为浮点数"""
        assert self.calc.divide(1, 3) == pytest.approx(0.3333, rel=1e-3)

    # ---- 边界条件 ----

    def test_divide_by_very_small_number(self):
        """除以极小数"""
        result = self.calc.divide(1, 0.0001)
        assert result == pytest.approx(10000.0)

    def test_divide_zero_numerator(self):
        """被除数为零"""
        assert self.calc.divide(0, 5) == 0.0

    # ---- 异常路径 ----

    def test_divide_by_zero_raises(self):
        """除以零抛出 ValueError"""
        with pytest.raises(ValueError, match="除数不能为零"):
            self.calc.divide(10, 0)


class TestCalculatorBatchAverage:
    """测试 Calculator.batch_average 方法"""

    def setup_method(self):
        self.calc = Calculator()

    def test_average_normal_list(self):
        """正常列表求平均"""
        assert self.calc.batch_average([1, 2, 3, 4, 5]) == 3.0

    def test_average_single_element(self):
        """单元素列表"""
        assert self.calc.batch_average([42]) == 42.0

    def test_average_negative_numbers(self):
        """含负数的列表"""
        assert self.calc.batch_average([-1, 1]) == 0.0

    def test_average_empty_list_raises(self):
        """空列表抛出 ValueError"""
        with pytest.raises(ValueError, match="列表不能为空"):
            self.calc.batch_average([])
```
