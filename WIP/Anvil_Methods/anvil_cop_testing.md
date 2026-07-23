# Testing Code of Practice
## Factory.AI + Anvil.works Testing Standard

**Version:** 1.0  
**Date:** January 31, 2026  
**Purpose:** Comprehensive testing standards for Factory Droid working with Anvil  
**Scope:** Pure logic testing + Uplink integration testing

---

## PHILOSOPHY

**Test to Enable Confidence, Not to Prevent Change**

Testing serves three purposes:
1. **Verification** - Prove code works as intended
2. **Documentation** - Show how code should be used
3. **Safety** - Catch regressions before they reach Anvil

**Core Principles:**
- Pure logic tested locally (fast, safe, iterative)
- Integration tested via Uplink (careful, verified, backed up)
- Tests document expected behavior
- All tests pass before backing up
- Failed tests stop development (fix first)

---

## TABLE OF CONTENTS

1. [Testing Levels](#1-testing-levels)
2. [Pure Logic Testing](#2-pure-logic-testing)
3. [Integration Testing via Uplink](#3-integration-testing-via-uplink)
4. [Test Structure Standards](#4-test-structure-standards)
5. [Test Naming Conventions](#5-test-naming-conventions)
6. [Test Execution Workflow](#6-test-execution-workflow)
7. [Test Documentation](#7-test-documentation)
8. [Backup Integration](#8-backup-integration)
9. [Common Testing Patterns](#9-common-testing-patterns)
10. [Troubleshooting Failed Tests](#10-troubleshooting-failed-tests)

---

## 1. TESTING LEVELS

### Level 1: Pure Logic Tests (Local Execution)

**What Gets Tested:**
- Business logic functions (calculations, validations, transformations)
- Data processing algorithms
- Pure Python functions (no Anvil dependencies)

**Where Tests Run:**
- Factory's local Python environment (via Factory Bridge)
- NO Anvil connection required
- NO Data Tables access
- NO server functions

**Speed:** Very fast (milliseconds)

**Risk:** Zero (cannot affect Anvil app)

**Frequency:** Continuous (every code change)

**Who Runs:** Factory Droid automatically

### Level 2: Integration Tests (Uplink Execution)

**What Gets Tested:**
- Server function integration
- Data Tables operations (via server functions)
- Anvil service integration (Users, Email, etc.)
- Multi-component interactions

**Where Tests Run:**
- Factory's local Python connected to Anvil via Uplink
- Accesses real Anvil app
- Reads/writes to Data Tables
- Calls server functions

**Speed:** Slower (seconds)

**Risk:** Medium (can affect Anvil app if not careful)

**Frequency:** After pure logic tests pass

**Who Runs:** Factory Droid via Uplink (after backup created)

**Safety Rule:** ALWAYS backup before integration tests

### Level 3: End-to-End Tests (Manual Verification)

**What Gets Tested:**
- Complete user workflows
- UI interactions
- Cross-feature integration
- User experience validation

**Where Tests Run:**
- Anvil app in browser
- Manual interaction by developer

**Speed:** Slowest (minutes)

**Risk:** Low (read-only verification)

**Frequency:** After work increment complete

**Who Runs:** Developer manually

---

## 2. PURE LOGIC TESTING

### Pure Logic Extraction Pattern

**Definition:** Business logic with zero Anvil dependencies

**Pure Function Characteristics:**
- ✅ No `import anvil` statements
- ✅ No Data Tables access
- ✅ No server function calls
- ✅ No side effects (file I/O, network, etc.)
- ✅ Deterministic (same inputs = same outputs)
- ✅ Only Python standard library + math/logic

**Example Pure Logic Module:**

```python
# order_logic.py - Pure business logic (NO Anvil imports)

from typing import Dict

class OrderLogicError(Exception):
    """Raised when order logic validation fails"""
    pass

def calculate_order_total(
    price: float,
    tax_rate: float,
    max_total: float
) -> Dict[str, float]:
    """
    Calculate order total with tax.
    
    Args:
        price: Base price (>= 0)
        tax_rate: Tax rate as decimal (>= 0)
        max_total: Maximum allowed total
    
    Returns:
        Dict with subtotal, tax, and total
    
    Raises:
        OrderLogicError: If validation fails
    """
    # Validation (fail fast)
    if price < 0:
        raise OrderLogicError("Price must be non-negative")
    
    if tax_rate < 0:
        raise OrderLogicError("Tax rate must be non-negative")
    
    # Calculation (pure logic)
    subtotal = price
    tax = subtotal * tax_rate
    total = subtotal + tax
    
    # Business rule validation
    if total > max_total:
        raise OrderLogicError(
            f"Total ${total:.2f} exceeds maximum ${max_total:.2f}"
        )
    
    return {
        "subtotal": subtotal,
        "tax": tax,
        "total": total
    }
```

**Why This Matters:**
- ✅ Factory can test locally (no Anvil connection)
- ✅ Tests run in milliseconds
- ✅ Zero risk to Anvil app
- ✅ Fast iteration
- ✅ Easy debugging

### Pure Logic Test Structure

**Test Module:** `test_order_logic.py`

```python
# test_order_logic.py - Pure logic tests (NO Anvil imports)

from order_logic import calculate_order_total, OrderLogicError

def test_valid_order():
    """Test: Valid order calculates correctly"""
    # Arrange
    price = 100.0
    tax_rate = 0.2
    max_total = 10000.0
    
    # Act
    result = calculate_order_total(price, tax_rate, max_total)
    
    # Assert
    assert result["subtotal"] == 100.0, f"Expected subtotal 100.0, got {result['subtotal']}"
    assert result["tax"] == 20.0, f"Expected tax 20.0, got {result['tax']}"
    assert result["total"] == 120.0, f"Expected total 120.0, got {result['total']}"
    
    print("✅ test_valid_order passed")

def test_negative_price():
    """Test: Negative price raises OrderLogicError"""
    # Arrange
    price = -1.0
    tax_rate = 0.2
    max_total = 10000.0
    
    # Act & Assert
    try:
        calculate_order_total(price, tax_rate, max_total)
        assert False, "Expected OrderLogicError to be raised"
    except OrderLogicError as e:
        assert "non-negative" in str(e).lower()
        print("✅ test_negative_price passed")

def test_exceeds_maximum():
    """Test: Total exceeding max raises OrderLogicError"""
    # Arrange
    price = 9000.0
    tax_rate = 0.5  # 50% tax
    max_total = 10000.0
    
    # Act & Assert
    try:
        calculate_order_total(price, tax_rate, max_total)
        assert False, "Expected OrderLogicError to be raised"
    except OrderLogicError as e:
        assert "exceeds maximum" in str(e).lower()
        print("✅ test_exceeds_maximum passed")

def test_zero_price():
    """Test: Zero price is valid"""
    # Arrange
    price = 0.0
    tax_rate = 0.2
    max_total = 10000.0
    
    # Act
    result = calculate_order_total(price, tax_rate, max_total)
    
    # Assert
    assert result["total"] == 0.0
    print("✅ test_zero_price passed")

def test_zero_tax_rate():
    """Test: Zero tax rate is valid"""
    # Arrange
    price = 100.0
    tax_rate = 0.0
    max_total = 10000.0
    
    # Act
    result = calculate_order_total(price, tax_rate, max_total)
    
    # Assert
    assert result["tax"] == 0.0
    assert result["total"] == 100.0
    print("✅ test_zero_tax_rate passed")

def run_all_tests():
    """
    Run all pure logic tests.
    Returns dict with pass/fail counts and error details.
    """
    results = {
        "passed": 0,
        "failed": 0,
        "errors": []
    }
    
    tests = [
        test_valid_order,
        test_negative_price,
        test_exceeds_maximum,
        test_zero_price,
        test_zero_tax_rate
    ]
    
    for test_func in tests:
        try:
            test_func()
            results["passed"] += 1
        except Exception as e:
            results["failed"] += 1
            results["errors"].append({
                "test": test_func.__name__,
                "error": str(e)
            })
            print(f"❌ {test_func.__name__} failed: {e}")
    
    return results

if __name__ == "__main__":
    print("Running pure logic tests...")
    print("-" * 50)
    
    results = run_all_tests()
    
    print("-" * 50)
    print(f"Results: {results['passed']} passed, {results['failed']} failed")
    
    if results['errors']:
        print("\nErrors:")
        for error in results['errors']:
            print(f"  {error['test']}: {error['error']}")
        exit(1)  # Exit with error code
    else:
        print("\n✅ All tests passed!")
        exit(0)  # Exit successfully
```

### Factory's Pure Logic Testing Workflow

```
1. Factory extracts business logic to pure function
2. Factory writes pure logic tests
3. Factory runs tests locally: python test_order_logic.py
4. IF all tests pass:
   ✅ Continue to integration layer
5. IF any tests fail:
   ❌ Fix logic
   ❌ Re-run tests
   ❌ Do NOT proceed until passing
```

---

## 3. INTEGRATION TESTING VIA UPLINK

### When Integration Tests Are Needed

**Use Cases:**
- Testing Data Tables operations
- Testing server function interactions
- Testing Anvil service integration (Users, Email)
- Testing multi-component workflows

**When NOT Needed:**
- Pure business logic (use pure logic tests)
- Simple calculations (use pure logic tests)
- Utility functions (use pure logic tests)

### Integration Test Safety Protocol

**MANDATORY STEPS:**

**BEFORE Writing Integration Tests:**
1. ✅ All pure logic tests passing
2. ✅ Backup created
3. ✅ Uplink connection verified
4. ✅ Dev log entry documenting intent

**DURING Integration Tests:**
1. ✅ Read before writing (verify current state)
2. ✅ Use test data (mark with source='Test')
3. ✅ Verify changes after writing
4. ✅ Log all operations

**AFTER Integration Tests:**
1. ✅ Clean up test data (delete test records)
2. ✅ Verify cleanup successful
3. ✅ Document results in dev log
4. ✅ Create backup if tests pass

### Integration Test Structure

**Integration Layer:** Wraps pure logic with Anvil services

```python
# server_orders.py - Anvil integration (thin wrapper)

import anvil.server
from anvil.tables import app_tables
import logging
from datetime import datetime

from .order_logic import calculate_order_total, OrderLogicError
from .constants import MAX_ORDER_TOTAL, DEFAULT_TAX_RATE

logger = logging.getLogger(__name__)

@anvil.server.callable
def create_order(customer_email: str, price: float):
    """
    Create order with calculated total.
    
    Args:
        customer_email: Customer's email address
        price: Base order price
    
    Returns:
        Order row object
    
    Raises:
        Exception: If validation fails or order creation fails
    """
    # Get authenticated user
    user = anvil.users.get_user()
    if not user:
        raise Exception("Authentication required")
    
    # Find customer (Data Tables operation)
    customer = app_tables.customers.get(
        instance_id=user,
        email=customer_email
    )
    if not customer:
        raise Exception(f"Customer {customer_email} not found")
    
    # Calculate order total (pure logic - already tested)
    try:
        order_calc = calculate_order_total(
            price=price,
            tax_rate=DEFAULT_TAX_RATE,
            max_total=MAX_ORDER_TOTAL
        )
    except OrderLogicError as e:
        logger.error(f"Order calculation failed: {e}")
        raise Exception(f"Invalid order: {e}")
    
    # Create order record (Data Tables operation)
    order = app_tables.orders.add_row(
        instance_id=user,
        customer=customer,
        subtotal=order_calc["subtotal"],
        tax=order_calc["tax"],
        total=order_calc["total"],
        status="pending",
        created_at=datetime.now()
    )
    
    logger.info(
        f"Order created: total=${order_calc['total']:.2f}",
        extra={"order_id": order.get_id(), "customer_email": customer_email}
    )
    
    return order
```

**Integration Test Module:** `test_orders_integration.py`

```python
# test_orders_integration.py - Integration tests via Uplink

import anvil.server
from anvil.tables import app_tables
import anvil.users
import logging
from datetime import datetime

logging.basicConfig(level=logging.INFO)
logger = logging.getLogger(__name__)

def setup_test_data():
    """Create test customer for integration tests"""
    user = anvil.users.get_user()
    if not user:
        raise Exception("Must be authenticated for integration tests")
    
    # Create test customer
    test_customer = app_tables.customers.add_row(
        instance_id=user,
        email="test_customer@example.com",
        first_name="Test",
        last_name="Customer",
        source="Test",  # Mark as test data
        created_at=datetime.now()
    )
    
    logger.info("Test customer created")
    return test_customer

def cleanup_test_data():
    """Delete all test data"""
    user = anvil.users.get_user()
    if not user:
        return
    
    # Delete test orders
    test_orders = app_tables.orders.search(
        instance_id=user,
        customer=app_tables.customers.get(
            instance_id=user,
            source="Test"
        )
    )
    for order in list(test_orders):
        order.delete()
    
    # Delete test customers
    test_customers = app_tables.customers.search(
        instance_id=user,
        source="Test"
    )
    for customer in list(test_customers):
        customer.delete()
    
    logger.info("Test data cleaned up")

def test_create_order_success():
    """Test: Creating valid order succeeds"""
    logger.info("TEST: test_create_order_success")
    
    # Setup
    test_customer = setup_test_data()
    
    try:
        # Act
        order = anvil.server.call(
            'create_order',
            customer_email="test_customer@example.com",
            price=100.0
        )
        
        # Assert
        assert order is not None, "Order should be created"
        assert order['subtotal'] == 100.0, f"Expected subtotal 100.0, got {order['subtotal']}"
        assert order['tax'] == 20.0, f"Expected tax 20.0, got {order['tax']}"
        assert order['total'] == 120.0, f"Expected total 120.0, got {order['total']}"
        assert order['status'] == "pending", f"Expected status 'pending', got {order['status']}"
        
        logger.info("✅ test_create_order_success passed")
        return True
        
    except Exception as e:
        logger.error(f"❌ test_create_order_success failed: {e}")
        return False
    
    finally:
        # Cleanup
        cleanup_test_data()

def test_create_order_invalid_customer():
    """Test: Creating order for non-existent customer fails"""
    logger.info("TEST: test_create_order_invalid_customer")
    
    try:
        # Act - Try to create order for non-existent customer
        try:
            order = anvil.server.call(
                'create_order',
                customer_email="nonexistent@example.com",
                price=100.0
            )
            logger.error("Expected exception but order was created")
            return False
        except Exception as e:
            # Assert - Should raise exception
            assert "not found" in str(e).lower(), f"Expected 'not found' error, got: {e}"
            logger.info("✅ test_create_order_invalid_customer passed")
            return True
    
    except Exception as e:
        logger.error(f"❌ test_create_order_invalid_customer failed: {e}")
        return False

def test_create_order_exceeds_maximum():
    """Test: Creating order exceeding max total fails"""
    logger.info("TEST: test_create_order_exceeds_maximum")
    
    # Setup
    test_customer = setup_test_data()
    
    try:
        # Act - Try to create order with price that exceeds maximum
        try:
            order = anvil.server.call(
                'create_order',
                customer_email="test_customer@example.com",
                price=20000.0  # Will exceed max after tax
            )
            logger.error("Expected exception but order was created")
            return False
        except Exception as e:
            # Assert - Should raise exception
            assert "invalid order" in str(e).lower(), f"Expected 'invalid order' error, got: {e}"
            logger.info("✅ test_create_order_exceeds_maximum passed")
            return True
    
    except Exception as e:
        logger.error(f"❌ test_create_order_exceeds_maximum failed: {e}")
        return False
    
    finally:
        # Cleanup
        cleanup_test_data()

def run_all_integration_tests():
    """
    Run all integration tests.
    Returns dict with pass/fail counts.
    """
    results = {
        "passed": 0,
        "failed": 0,
        "tests": []
    }
    
    tests = [
        test_create_order_success,
        test_create_order_invalid_customer,
        test_create_order_exceeds_maximum
    ]
    
    logger.info("=" * 60)
    logger.info("RUNNING INTEGRATION TESTS VIA UPLINK")
    logger.info("=" * 60)
    
    for test_func in tests:
        test_name = test_func.__name__
        try:
            passed = test_func()
            if passed:
                results["passed"] += 1
                results["tests"].append({"name": test_name, "status": "PASSED"})
            else:
                results["failed"] += 1
                results["tests"].append({"name": test_name, "status": "FAILED"})
        except Exception as e:
            results["failed"] += 1
            results["tests"].append({
                "name": test_name,
                "status": "ERROR",
                "error": str(e)
            })
            logger.error(f"Test {test_name} threw exception: {e}")
    
    logger.info("=" * 60)
    logger.info(f"RESULTS: {results['passed']} passed, {results['failed']} failed")
    logger.info("=" * 60)
    
    return results

if __name__ == "__main__":
    # Connect to Anvil via Uplink
    import os
    UPLINK_KEY = os.environ.get('ANVIL_UPLINK_KEY')
    if not UPLINK_KEY:
        print("ERROR: ANVIL_UPLINK_KEY environment variable not set")
        exit(1)
    
    logger.info("Connecting to Anvil via Uplink...")
    anvil.server.connect(UPLINK_KEY)
    logger.info("Connected successfully")
    
    # Run tests
    results = run_all_integration_tests()
    
    # Exit with appropriate code
    exit(0 if results['failed'] == 0 else 1)
```

---

## 4. TEST STRUCTURE STANDARDS

### Test Function Template

```python
def test_[feature]_[scenario]():
    """
    Test: [Clear description of what's being tested]
    
    Given: [Initial conditions]
    When: [Action performed]
    Then: [Expected outcome]
    """
    # Arrange - Set up test data and conditions
    input_value = ...
    expected_output = ...
    
    # Act - Execute the code being tested
    result = function_under_test(input_value)
    
    # Assert - Verify the outcome
    assert result == expected_output, f"Expected {expected_output}, got {result}"
    
    print(f"✅ test_[feature]_[scenario] passed")
```

### Test Module Structure

```python
# test_[module_name].py

# 1. Imports
from [module] import [functions_to_test], [exceptions_to_test]
import logging

# 2. Setup logging (for integration tests)
logging.basicConfig(level=logging.INFO)
logger = logging.getLogger(__name__)

# 3. Helper functions (if needed)
def setup_test_data():
    """Setup function for integration tests"""
    pass

def cleanup_test_data():
    """Cleanup function for integration tests"""
    pass

# 4. Test functions
def test_case_1():
    """Test: Description"""
    pass

def test_case_2():
    """Test: Description"""
    pass

# 5. Test runner
def run_all_tests():
    """Run all tests and return results"""
    results = {"passed": 0, "failed": 0, "errors": []}
    
    tests = [test_case_1, test_case_2]
    
    for test_func in tests:
        try:
            test_func()
            results["passed"] += 1
        except Exception as e:
            results["failed"] += 1
            results["errors"].append({
                "test": test_func.__name__,
                "error": str(e)
            })
    
    return results

# 6. Main execution
if __name__ == "__main__":
    results = run_all_tests()
    print(f"Results: {results['passed']} passed, {results['failed']} failed")
    exit(0 if results['failed'] == 0 else 1)
```

---

## 5. TEST NAMING CONVENTIONS

### Test Function Names

**Format:** `test_[feature]_[scenario]`

**Examples:**
- `test_calculate_order_valid_input` - Happy path test
- `test_calculate_order_negative_price` - Error condition
- `test_calculate_order_exceeds_maximum` - Business rule violation
- `test_create_customer_duplicate_email` - Constraint violation
- `test_authenticate_user_invalid_password` - Authentication failure

**Rules:**
- Start with `test_` (enables test discovery)
- Use underscores (snake_case)
- Be descriptive (what and why)
- Group related tests (same prefix)

### Test Module Names

**Format:** `test_[module_name].py`

**Examples:**
- `test_order_logic.py` - Pure logic tests
- `test_orders_integration.py` - Integration tests
- `test_contact_validation.py` - Validation tests
- `test_authentication.py` - Auth tests

### Test Data Naming

**Mark Test Data:**
- Add `source='Test'` field to test Data Tables rows
- Use `test_` prefix for test user emails: `test_user@example.com`
- Use descriptive names: `test_customer`, `test_order`, `test_product`

---

## 6. TEST EXECUTION WORKFLOW

### Pure Logic Test Execution

**When:** Immediately after writing/modifying pure logic code

**How Factory Executes:**
```bash
# Run via Factory Bridge
python test_order_logic.py
```

**Success Criteria:**
- All tests pass
- Exit code 0
- No exceptions raised

**On Failure:**
- Fix logic immediately
- Re-run tests
- Do NOT proceed until passing

### Integration Test Execution

**When:** After pure logic tests pass AND backup created

**How Factory Executes:**
```bash
# 1. Verify Uplink connection
python -c "import anvil.server; import os; anvil.server.connect(os.environ['ANVIL_UPLINK_KEY']); print('Connected')"

# 2. Run integration tests
python test_orders_integration.py
```

**Success Criteria:**
- All tests pass
- Test data cleaned up
- Exit code 0
- No exceptions raised

**On Failure:**
- Review error messages
- Check Anvil app logs
- Fix integration code
- Restore from backup if needed
- Re-run tests

### Complete Test Workflow

```
1. Factory writes pure logic
2. Factory writes pure logic tests
3. Factory runs pure logic tests locally
   ├─ Pass → Continue
   └─ Fail → Fix and retry

4. Factory creates BACKUP (before integration)
5. Factory writes integration layer
6. Factory writes integration tests
7. Factory runs integration tests via Uplink
   ├─ Pass → Create backup (after success)
   └─ Fail → Restore backup, fix, retry

8. Factory updates dev log with test results
9. Factory continues to next work increment
```

---

## 7. TEST DOCUMENTATION

### Test Results in Dev Log

**After Every Test Run, Document:**

```markdown
### Tests
**Pure Logic Tests:**
- ✅ test_calculate_order_valid_input - passed
- ✅ test_calculate_order_negative_price - passed
- ✅ test_calculate_order_exceeds_maximum - passed
- ✅ test_calculate_order_zero_price - passed
Total: 4/4 passed

**Integration Tests:**
- ✅ test_create_order_success - passed
- ✅ test_create_order_invalid_customer - passed
- ✅ test_create_order_exceeds_maximum - passed
Total: 3/3 passed

**Overall:** ✅ All tests passing (7/7)
```

### Failed Test Documentation

```markdown
### Tests
**Pure Logic Tests:**
- ✅ test_calculate_order_valid_input - passed
- ❌ test_calculate_order_negative_price - FAILED
  Error: Expected OrderLogicError but none raised
  Fix: Added validation check for negative price
  Retest: ✅ Now passing

**Status:** All tests passing after fix
```

---

## 8. BACKUP INTEGRATION

### Tests and Backups Relationship

**Backup Triggers Based on Tests:**

**1. Before Integration Tests (Safety Checkpoint):**
```bash
# Tests passing locally → Create backup
./backup_repo.sh "before-integration-tests-[feature]"
```

**2. After Integration Tests Pass (Known Good State):**
```bash
# Integration tests passing → Create backup
./backup_repo.sh "[feature]-tests-passing"
```

**3. Never Backup Failed Tests:**
- ❌ Do NOT backup if any tests failing
- ❌ Do NOT backup incomplete test suites
- ✅ Only backup "known good" states

### Dev Log Test Entries

**Standard Test Entry Format:**

```markdown
## 2026-02-01 14:30 - Order Logic Implementation Complete

### Backup
- **Name:** 2026-02-01_14-30_order-logic-complete
- **Path:** /home/user/Mybizz-backups/2026-02-01_14-30_order-logic-complete

### Work Completed
- Implemented calculate_order_total() pure logic function
- Added OrderLogicError exception class
- Created constants.py with MAX_ORDER_TOTAL, DEFAULT_TAX_RATE

### Tests - Pure Logic
- ✅ test_calculate_order_valid_input
- ✅ test_calculate_order_negative_price
- ✅ test_calculate_order_exceeds_maximum
- ✅ test_calculate_order_zero_price
- ✅ test_calculate_order_zero_tax_rate
**Result:** 5/5 passed

### Tests - Integration
- Not yet implemented (pure logic only)

### Next Steps
- [ ] Create server_orders.py integration layer
- [ ] Write integration tests
- [ ] Test via Uplink
```

---

## 9. COMMON TESTING PATTERNS

### Pattern 1: Valid Input (Happy Path)

```python
def test_function_valid_input():
    """Test: Function works with valid input"""
    # Arrange
    valid_input = ...
    expected_output = ...
    
    # Act
    result = function(valid_input)
    
    # Assert
    assert result == expected_output
    print("✅ test_function_valid_input passed")
```

### Pattern 2: Invalid Input (Error Handling)

```python
def test_function_invalid_input():
    """Test: Function raises appropriate error for invalid input"""
    # Arrange
    invalid_input = ...
    
    # Act & Assert
    try:
        function(invalid_input)
        assert False, "Expected exception but none raised"
    except ExpectedException as e:
        assert "expected message" in str(e).lower()
        print("✅ test_function_invalid_input passed")
```

### Pattern 3: Boundary Conditions

```python
def test_function_boundary_zero():
    """Test: Function handles zero value"""
    assert function(0) == expected_zero_result
    print("✅ test_function_boundary_zero passed")

def test_function_boundary_minimum():
    """Test: Function handles minimum value"""
    assert function(min_value) == expected_min_result
    print("✅ test_function_boundary_minimum passed")

def test_function_boundary_maximum():
    """Test: Function handles maximum value"""
    assert function(max_value) == expected_max_result
    print("✅ test_function_boundary_maximum passed")
```

### Pattern 4: Data Tables Integration

```python
def test_create_record():
    """Test: Creating Data Tables record succeeds"""
    # Setup
    user = anvil.users.get_user()
    
    # Act
    record = app_tables.table_name.add_row(
        instance_id=user,
        field1="value1",
        source="Test"  # Mark as test data
    )
    
    try:
        # Assert
        assert record is not None
        assert record['field1'] == "value1"
        print("✅ test_create_record passed")
    finally:
        # Cleanup
        record.delete()
```

### Pattern 5: Multi-Tenant Security

```python
def test_cannot_access_other_tenant_data():
    """Test: Data Tables properly isolates tenants"""
    user = anvil.users.get_user()
    
    # Try to access records from another tenant
    other_records = app_tables.table_name.search(
        instance_id=None  # Should return empty
    )
    
    assert len(list(other_records)) == 0, "Should not access other tenant's data"
    print("✅ test_cannot_access_other_tenant_data passed")
```

---

## 10. TROUBLESHOOTING FAILED TESTS

### Debugging Process

**1. Read Error Message:**
```python
# Error message shows:
# - Which test failed
# - What assertion failed
# - Expected vs actual values
```

**2. Check Test Logic:**
```python
# Is the test correct?
# - Are assertions valid?
# - Is test data correct?
# - Are expectations reasonable?
```

**3. Check Code Logic:**
```python
# Is the code correct?
# - Does it handle edge cases?
# - Are calculations correct?
# - Are validations correct?
```

**4. Check Integration:**
```python
# For integration tests:
# - Is Uplink connected?
# - Are Data Tables accessible?
# - Are server functions callable?
```

### Common Failure Patterns

**Pattern 1: Assertion Mismatch**
```
❌ AssertionError: Expected 100.0, got 120.0
```
**Fix:** Check calculation logic, verify test expectations

**Pattern 2: Exception Not Raised**
```
❌ AssertionError: Expected OrderLogicError but none raised
```
**Fix:** Add validation check in code

**Pattern 3: Wrong Exception Type**
```
❌ ValueError raised instead of OrderLogicError
```
**Fix:** Use correct exception type

**Pattern 4: Uplink Connection Failed**
```
❌ Exception: Not connected to Anvil
```
**Fix:** Verify Uplink key, check connection

**Pattern 5: Data Tables Access Error**
```
❌ Exception: Authentication required
```
**Fix:** Verify user authenticated before test

### Recovery Steps

**If Pure Logic Tests Fail:**
1. Fix logic immediately
2. Re-run tests
3. Do NOT proceed until passing

**If Integration Tests Fail:**
1. Review Anvil app logs
2. Check Data Tables state
3. Fix integration code
4. Consider restoring from backup if needed
5. Re-run tests
6. Only create backup after tests pass

---

## TESTING CHECKLIST

### Before Writing Tests
- [ ] Pure logic extracted from Anvil code
- [ ] Functions have clear inputs and outputs
- [ ] Edge cases identified
- [ ] Error conditions documented

### Pure Logic Tests
- [ ] Test file created: `test_[module].py`
- [ ] All happy path cases covered
- [ ] All error conditions covered
- [ ] All boundary conditions covered
- [ ] Tests run locally (no Anvil dependency)
- [ ] All tests passing
- [ ] Exit code 0

### Integration Tests
- [ ] Backup created BEFORE writing tests
- [ ] Uplink connection verified
- [ ] Test data marked with `source='Test'`
- [ ] Setup/cleanup functions created
- [ ] All integration scenarios covered
- [ ] Tests cleanup after themselves
- [ ] All tests passing via Uplink
- [ ] Backup created AFTER tests pass

### Documentation
- [ ] Test results documented in dev log
- [ ] Pass/fail counts recorded
- [ ] Failed tests documented with fixes
- [ ] Backup linked to test results
- [ ] Next steps identified

---

## CONCLUSION

**Testing enables confident, rapid development:**

**Pure Logic Tests:**
- ✅ Fast (milliseconds)
- ✅ Safe (no Anvil risk)
- ✅ Iterative (test-driven development)

**Integration Tests:**
- ✅ Careful (backup first)
- ✅ Verified (check results)
- ✅ Cleaned (remove test data)

**All Tests:**
- ✅ Documented (dev log entries)
- ✅ Backed up (only when passing)
- ✅ Repeatable (consistent results)

**Remember:**
- Pure logic tested locally (Factory runs freely)
- Integration tested via Uplink (Factory runs carefully)
- Backup before integration tests (safety net)
- Backup after tests pass (known good state)
- Document everything (dev log)

**Tests are not obstacles - they are enablers.**

---

**END OF TESTING CODE OF PRACTICE v1.0**
