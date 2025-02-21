# The Art of Testing Python Code

Testing transforms your code from a fragile house of cards into a robust, maintainable fortress by systematically validating its behavior and documenting its intent.

Admittedly, testing is not something I've fully embraced, I don't have (yet) a single codebase that has 90+% test coverage; however, the more I've worked on projects where more than one person writes the code, the more I understand their necessity. At the end of the day, just like with [typing](/python/typing/intro/#configuring-mypy), it's something you need to force yourself to do once, and after that it will become a natural part of coding.

## Testing in the wild

Testing is pretty much an unquestionable industry standard. Any codebase that will be reused strives to have code coverage of at least 90% (meaning that 90% of lines of code have a test case explicitly testing them).

Perhaps the most famous example of a well tested codebase is [SQLite](https://www.sqlite.org/index.html), a SQL based db used by Adobe, Google (Android and Chrome), Apple (iOS), Dropbox, and many other giants. SQLite library contains 155 800 lines of source code in C (excluding blanks and comments). The test suite for SQLite has 92 053 100 lines of code, i.e. 590 lines of testing for every line of code.

> The reliability and robustness of SQLite is achieved in part by thorough and careful testing.[[source]](https://www.sqlite.org/testing.html)

Python code can be tested using pytest library. The rest of this page documents basic use.

## Prerequisites

- Python 3.7+
- Understanding of Python functions and classes

```bash
source .venv/bin/activate
uv add pytest typing-extensions mypy
```

## Testing Fundamentals

### 1. Unit Tests: The Foundation

Unit tests validate individual components in isolation. They are your first line of defense.

```python
from typing import Any
from decimal import Decimal


class PricingEngine:
    """Handles product pricing calculations."""

    def calculate_discount(self, price: Decimal, percentage: Decimal) -> Decimal:
        """Calculate discounted price.

        Args:
            price: Original price
            percentage: Discount percentage (0-100)

        Returns:
            Discounted price

        Raises:
            ValueError: If percentage is not between 0 and 100
        """
        if not 0 <= percentage <= 100:
            raise ValueError("Percentage must be between 0 and 100")
        return price * (1 - percentage / 100)


def test_pricing_engine() -> None:
    """Demonstrate comprehensive unit testing."""
    engine = PricingEngine()

    # Happy path
    assert engine.calculate_discount(Decimal("100.00"), Decimal("20.00")) == Decimal(
        "80.00"
    )

    # Edge cases
    assert engine.calculate_discount(Decimal("100.00"), Decimal("0.00")) == Decimal(
        "100.00"
    )
    assert engine.calculate_discount(Decimal("100.00"), Decimal("100.00")) == Decimal(
        "0.00"
    )

    # Error cases
    try:
        engine.calculate_discount(Decimal("100.00"), Decimal("101.00"))
        assert False, "Should raise ValueError"
    except ValueError as e:
        assert str(e) == "Percentage must be between 0 and 100"
```

### 2. Integration Tests: Component Harmony

Integration tests verify that components work together correctly. They catch interface mismatches and data flow issues.

```python
from typing import Optional
from dataclasses import dataclass
from decimal import Decimal


@dataclass
class Product:
    id: str
    name: str
    price: Decimal


class ProductDatabase:
    def __init__(self) -> None:
        self._products: dict[str, Product] = {}

    def add(self, product: Product) -> None:
        self._products[product.id] = product

    def get(self, product_id: str) -> Optional[Product]:
        return self._products.get(product_id)


class PricingService:
    def __init__(self, db: ProductDatabase, engine: PricingEngine) -> None:
        self.db = db
        self.engine = engine

    def apply_discount(self, product_id: str, discount: Decimal) -> Optional[Product]:
        """Apply discount to product price.

        Args:
            product_id: Product identifier
            discount: Discount percentage

        Returns:
            Updated product or None if not found
        """
        product = self.db.get(product_id)
        if not product:
            return None

        discounted_price = self.engine.calculate_discount(product.price, discount)
        return Product(product.id, product.name, discounted_price)


def test_pricing_service_integration() -> None:
    """Demonstrate integration testing."""
    # Setup components
    db = ProductDatabase()
    engine = PricingEngine()
    service = PricingService(db, engine)

    # Prepare test data
    original_product = Product("PROD1", "Test Product", Decimal("100.00"))
    db.add(original_product)

    # Test integrated flow
    discounted_product = service.apply_discount("PROD1", Decimal("20.00"))
    assert discounted_product is not None
    assert discounted_product.price == Decimal("80.00")

    # Test error handling
    assert service.apply_discount("NONEXISTENT", Decimal("20.00")) is None
```

### 3. Functional Tests: User Perspective

Functional tests validate complete features from a user's perspective. They ensure the system works as a whole.

```python
from fastapi import FastAPI, HTTPException
from fastapi.testclient import TestClient
from decimal import Decimal
from typing import Dict, Any

app = FastAPI()
db = ProductDatabase()
engine = PricingEngine()
service = PricingService(db, engine)


@app.post("/products/{product_id}/discount")
async def apply_discount(
    product_id: str, discount_percentage: Decimal
) -> Dict[str, Any]:
    """Apply discount to product."""
    result = service.apply_discount(product_id, discount_percentage)
    if not result:
        raise HTTPException(status_code=404, detail="Product not found")
    return {
        "id": result.id,
        "name": result.name,
        "original_price": str(result.price),
        "discount_percentage": str(discount_percentage),
        "final_price": str(result.price),
    }


def test_discount_api() -> None:
    """Demonstrate functional testing of the API."""
    client = TestClient(app)

    # Setup test data
    db.add(Product("PROD1", "Test Product", Decimal("100.00")))

    # Test successful discount
    response = client.post("/products/PROD1/discount?discount_percentage=20.0")
    assert response.status_code == 200
    data = response.json()
    assert data["final_price"] == "80.00"

    # Test error handling
    response = client.post("/products/NONEXISTENT/discount?discount_percentage=20.0")
    assert response.status_code == 404
```

## Testing Best Practices

1. Test Organization

    - One test file per source file
    - Group related tests in classes
    - Name tests descriptively

1. Test Coverage

    - Happy path: Normal operation
    - Edge cases: Boundary conditions
    - Error cases: Expected failures
    - Security: Input validation

1. Performance

    - Use appropriate test scopes
    - Mock expensive operations
    - Parallelize test execution

## Common Pitfalls

1. Incomplete Testing

    ```python
    # BAD: Only testing happy path
    def test_incomplete():
        assert calculate_discount(100, 20) == 80


    # GOOD: Testing all scenarios
    def test_complete():
        assert calculate_discount(100, 20) == 80  # Happy path
        assert calculate_discount(100, 0) == 100  # Edge case
        with pytest.raises(ValueError):  # Error case
            calculate_discount(100, -1)
    ```

1. Brittle Tests

    ```python
    # BAD: Testing implementation details
    def test_brittle(mocker):
        mocker.spy(service, "_internal_calculation")
        service.process()
        assert service._internal_calculation.called


    # GOOD: Testing observable behavior
    def test_robust():
        result = service.process()
        assert result.status == "success"
    ```

1. Poor Isolation

    ```python
    # BAD: Shared state between tests
    total = 0


    def test_shared_state1():
        global total
        total += 1
        assert total == 1


    # GOOD: Isolated tests
    def test_isolated1():
        calculator = Calculator()
        assert calculator.add(1) == 1
    ```

## Resources

- [pytest Documentation](https://docs.pytest.org/)
- [Testing Best Practices](https://docs.python-guide.org/writing/tests/)
- [Property-Based Testing](https://hypothesis.works/)
