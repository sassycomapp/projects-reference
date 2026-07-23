This is a high-caliber document that perfectly captures the "Pro-level" architecture needed for a production Anvil app. It moves beyond just writing code that works to writing code that is observable and resilient.

Review of Mybizz Anvil / Python Coding Code of Practice based on industry standards:

1. Strengths & Alignment
Fail Fast/Fail Safely Integration: You’ve successfully woven the "Fail Fast in Dev / Fail Safely in Prod" philosophy throughout every section.

Traceback Preservation: Section 8 correctly identifies the nuance between raise and raise ... from, which is the #1 way developers accidentally destroy debugging data.

The "Gold Standard" Examples: Providing concrete examples of how these rules look in a real Anvil Server and Client module makes this document actionable, not just theoretical.

Decoupling Logic: The "Pure Logic Extraction" pattern is perhaps the most important architectural advice in the doc. It makes testing fast and prevents the "Anvil Sprawl" (where logic is scattered across 50 forms).

2. Critical Observations
Logging vs. Print: Forbidding print() in server code is a bold but necessary move for production. Using the 5-tier system ensures you can filter out the "noise" when looking for real errors.

Specific Exceptions: Your rule against except: pass and bare except: is the correct implementation of the "Fail Fast" rule. It forces the developer to acknowledge what they are catching.

Global Error Handling: Using set_default_error_handling acts as your "safety net" to ensure that even if a developer forgets a try-except, the system still fails safely and logs the event.

3. Suggested Minor Additions
While the document is nearly perfect, you might consider adding these two "Micro-Rules":

Schema Validation: For server functions receiving complex dictionaries/JSON from the client, suggest a validation step (like pydantic or simple if checks) as part of the "Fail Fast" input validation.

Naming Conventions: Briefly mention that snake_case is standard for functions/variables and PascalCase for classes to keep the repo visually consistent with the Python standard library.

Summary Checklist for your Team
Before any code is merged, it should pass this "Rule of 5" from your document:

Contract: Does it have a Docstring with Args and Returns?

Type-Safe: Are the inputs and outputs Type Hinted?

Noisy: Does it use logging instead of print?

Resilient: If it uses try-except, is it catching a specific error?

Pure: Is the core math/decision logic separated from the UI/Data Table calls?


clean, authoritative, Anvil-specific Coding Code of Practice, ready to drop into your repo or internal docs.

Anvil / Python Coding Code of Practice
Core Philosophy

Fail Fast in Development. Fail Safely in Production.
Code must be readable, testable, observable, and resilient across Anvil’s client–server boundary.

1. Comprehensive Docstrings — The Contract

Every non-trivial function must have a docstring that defines expectations, not just behavior.

Rules

Use triple-quoted docstrings

Describe intent, inputs, outputs, and failure modes

Prefer Google or NumPy style

Why

Enables IDE hints and auto-documentation

Makes server APIs self-describing

Acts as a contract between UI and logic

Anvil Standard

Server Modules must be heavily doc-stringed

Treat server functions as public APIs

def calculate_total(price: float, tax: float) -> float:
    """
    Calculate the final price including tax.


    Args:
        price (float): Base price before tax.
        tax (float): Tax rate as a decimal (e.g. 0.2).


    Returns:
        float: Final price including tax.


    Raises:
        ValueError: If price or tax is negative.
    """
2. Meaningful Inline Comments — The “Why”, Not the “What”
Rules

Never explain obvious code

Explain intent, constraints, or non-obvious decisions

Why

Code explains how. Comments explain why.

Good
# API returns cents, not dollars — convert before storing
amount = api_value / 100
Bad
x = x + 1  # increment x
3. Five-Tier Logging — The Black Box

print() is forbidden in server code.

Required Logging Levels

DEBUG – internal state, dev only

INFO – normal operations

WARNING – unexpected but recoverable

ERROR – operation failed

CRITICAL – system integrity at risk

Why

Enables noise control

Allows forensic debugging after failure

Essential for production Anvil apps

Anvil Rule

All server failures must be logged

Logs must contain enough context to reproduce the issue

4. Specific try / except Blocks — Fail Safely
Rules

❌ Never use except:

Catch only expected exceptions

Let unexpected errors crash loudly in development

Why

Bare except hides bugs

Preserves meaningful tracebacks

try:
    result = 10 / user_input
except ZeroDivisionError:
    return "Cannot divide by zero"
5. Type Hinting — Early Bug Detection

All public functions must use type annotations.

Why

Catches bugs before runtime

Improves IDE autocomplete

Documents intent without comments

def get_user(id: int) -> dict:
    ...
Anvil Tip

Type hints are especially valuable for:

Server APIs

Data Table access

Cross-module calls

6. Single Source of Truth for Constants
Rules

No hard-coded URLs, API keys, or magic strings

No duplicated literals across files

Best Practice

Use a dedicated constants.py module

Use Anvil App Secrets for sensitive data

API_TIMEOUT_SECONDS = 30
DEFAULT_ROLE = "user"
7. Decouple Logic from UI
Rules

UI code handles presentation only

Business logic lives in:

Client Modules, or

Server Modules (preferred)

Why

Enables testing

Prevents duplicated logic

Keeps UI readable

Anvil Rule

Button click handlers should call functions, not contain logic.

8. Traceback Preservation Rules
1️⃣ Never use except: pass

Swallows errors and destroys debugging information.

2️⃣ Re-raise with raise (no arguments)
try:
    do_something()
except ValueError as e:
    logging.error(f"Validation error: {e}")
    raise  # preserves original traceback
3️⃣ Use exception chaining (raise … from)
try:
    connect_to_db()
except ConnectionError as e:
    raise DatabaseAccessError("Service unavailable") from e

This preserves the root cause in the traceback.

9. Anvil-Specific Rule: Client–Server Visibility

Server crashes may appear “silent” on the UI.

Mandatory Practices

Keep App Logs open in the Anvil IDE

Log full tracebacks server-side

Never guess or auto-repair failed database writes

If a critical operation fails, stop execution. Corrupt data is worse than no data.

10. Global Error Handling — Last Line of Defense

All apps should register a global client-side error handler.

Purpose

Catch uncaught errors

Log full tracebacks

Show safe user messages

import anvil
import anvil.server


def global_error_handler(err):
    # Log full traceback server-side
    anvil.server.call('log_error_to_server', repr(err))


    # User-safe message
    alert(
        "An unexpected error occurred. Please try again.",
        title="System Error"
    )


# Register handler
anvil.set_default_error_handling(global_error_handler)
Why This Works

Developers get full diagnostics

Users see a clean UI

No traceback goes unnoticed

Final Rule Summary

Readable over clever

Explicit over implicit

Fail fast in dev

Fail safely in prod

Logs over prints

Logic outside UI

Tracebacks are sacred

---

Below is a “Gold Standard” Anvil Server Function that explicitly applies every rule from your Code of Practice.
This is real, runnable Anvil server code — not pseudocode.

Gold Standard Server Function (Anvil)
Scenario

Create or update an order total safely, with:

Docstring contract

Type hints

Constants

Logging (5-tier)

Specific exception handling

Traceback preservation

Fail-fast / fail-safe behavior

Clear UI–server boundary

constants.py (Single Source of Truth)
# Server Module: constants.py


MAX_ORDER_TOTAL = 10_000.00
DEFAULT_TAX_RATE = 0.2
server_orders.py (Gold Standard Server Module)
# Server Module: server_orders.py
        extra={"order_id": order_id}
    )


    # ---- Input Validation (Fail Fast) ----
    if price < 0:
        raise OrderValidationError("Price must be non-negative")


    if tax_rate < 0:
        raise OrderValidationError("Tax rate must be non-negative")


    subtotal: float = price
    tax: float = subtotal * tax_rate
    total: float = subtotal + tax


    if total > MAX_ORDER_TOTAL:
        raise OrderValidationError(
            f"Order total exceeds maximum allowed ({MAX_ORDER_TOTAL})"
        )


    logger.debug(
        "Calculated totals",
        extra={
            "subtotal": subtotal,
            "tax": tax,
            "total": total
        }
    )


    # ---- Database Operation ----
    try:
        order_row = app_tables.orders.get(id=order_id)


        if order_row is None:
            raise LookupError(f"Order {order_id} not found")


        order_row.update(
            subtotal=subtotal,
            tax=tax,
            total=total
        )


    except tables.TableError as e:
        # Expected infrastructure failure → log + re-raise
        logger.error(
            "Database write failed",
            exc_info=True,
            extra={"order_id": order_id}
        )
        raise RuntimeError("Failed to store order total") from e


    except Exception:
        # Unexpected failure → preserve traceback
        logger.critical(
            "Unexpected error while updating order",
            exc_info=True,
            extra={"order_id": order_id}
        )
        raise


    logger.info(
        "Order total successfully stored",
        extra={"order_id": order_id, "total": total}
    )


    return {
        "subtotal": subtotal,
        "tax": tax,
        "total": total
    }
Why this is “Gold Standard”
✅ Docstring = Contract

Args, Returns, Raises clearly defined

Server API is self-documenting

✅ Type Hinting

Early bug detection

IDE support

Clear intent

✅ Constants

No magic numbers

Single source of truth

✅ Five-Tier Logging

INFO for lifecycle

DEBUG for calculations

ERROR for expected failures

CRITICAL for unknown failures

✅ Specific try / except

Catches only what is expected

Lets the unexpected fail loudly

✅ Traceback Preservation

raise preserves stack

raise ... from e chains exceptions

✅ Fail Fast / Fail Safely

Invalid data → immediate exception

Infrastructure failure → safe message + full log

No silent corruption

✅ Anvil-Appropriate

Server-only logic

Clean UI–server separation

Callable API boundary

How the UI should call this (correct pattern)
try:
    result = anvil.server.call(
        'calculate_and_store_order_total',
        order_id=123,
        price=99.99
    )
    alert(f"Total: {result['total']}")
except anvil.server.AnvilWrappedError as e:
    alert(str(e), title="Order Error")
	

---

Below is a “Gold Standard” Anvil Client Module that pairs cleanly with your server-side gold standard.
It follows the same rules, but adapted correctly for client-side responsibilities.

This is real, production-ready Anvil code.

Gold Standard Client Module (Anvil)
Purpose of a Client Module (Code of Practice Reminder)

A Client Module:

Contains UI-adjacent logic, not UI rendering

Coordinates server calls

Handles expected user-facing errors

Never performs business-critical logic

Is easy to test and reuse across Forms

Scenario

Validate user input

Call the server API

Translate server errors into user-safe messages

Preserve tracebacks via global handler

client_orders.py (Client Module)
# Client Module: client_orders.py


import anvil.server
import logging
from typing import Dict


logger = logging.getLogger(__name__)




class UserInputError(Exception):
    """Raised when user input is invalid on the client side."""




def submit_order_total(
    order_id: int,
    price_text: str
) -> Dict[str, float]:
    """
    Validate user input and submit an order total calculation request
    to the server.


    This function performs lightweight client-side validation,
    delegates business logic to the server, and returns the
    calculated totals.


    Args:
        order_id (int): ID of the order being updated.
        price_text (str): Raw price input from a UI TextBox.


    Returns:
        dict: Dictionary containing subtotal, tax, and total.


    Raises:
        UserInputError: If the user input cannot be parsed.
        anvil.server.AnvilWrappedError: If the server rejects the request.
    """


    logger.debug(
        "Submitting order total",
        extra={"order_id": order_id, "price_text": price_text}
    )


    # ---- Client-side Validation (Fail Fast, UI-Friendly) ----
    try:
        price: float = float(price_text)
    except ValueError:
        raise UserInputError("Price must be a valid number")


    if price < 0:
        raise UserInputError("Price cannot be negative")


    # ---- Server Call (Business Logic Boundary) ----
    try:
        result: Dict[str, float] = anvil.server.call(
            'calculate_and_store_order_total',
            order_id=order_id,
            price=price
        )


    except anvil.server.TimeoutError:
        logger.warning(
            "Server timeout while submitting order",
            extra={"order_id": order_id}
        )
        raise


    except anvil.server.AnvilWrappedError as e:
        # Expected business-rule or validation failure
        logger.info(
            "Server rejected order submission",
            extra={"order_id": order_id, "error": str(e)}
        )
        raise


    except Exception:
        # Unexpected client-side failure → preserve traceback
        logger.error(
            "Unexpected client error",
            exc_info=True,
            extra={"order_id": order_id}
        )
        raise


    logger.info(
        "Order submission successful",
        extra={"order_id": order_id, "total": result.get("total")}
    )


    return result
Why this is “Gold Standard” (Client Edition)
✅ Docstring = Contract

Explicit inputs, outputs, and error behavior

Clear UI ↔ logic boundary

✅ Type Hinting

Especially important when passing data across modules

Makes server contracts obvious

✅ No UI Code

No alert()

No component references

Fully reusable from any Form

✅ Fail Fast (Client-Side)

Bad input is rejected before server call

UserInputError is intentional and meaningful

✅ Fail Safely (Server-Side)

Business rules enforced on the server

Server decides what is valid or not

✅ Logging (Client-Appropriate)

DEBUG for input

INFO for success/rejection

WARNING for timeouts

ERROR for unexpected failures

✅ Tracebacks Preserved

No swallowed exceptions

Unexpected errors bubble to global handler

Correct Usage from a Form (Pattern)
from ..client_orders import submit_order_total, UserInputError


def submit_button_click(self, **event_args):
    try:
        result = submit_order_total(
            order_id=self.order_id,
            price_text=self.price_box.text
        )
        self.total_label.text = f"{result['total']:.2f}"


    except UserInputError as e:
        alert(str(e), title="Input Error")


    except anvil.server.AnvilWrappedError as e:
        alert(str(e), title="Order Error")

Notice:

Form handles presentation only

No business logic

No duplication

Clean responsibility separation

Mental Model (Very Important)
Form
  ↓
Client Module  ← UI validation, coordination
  ↓
Server Module  ← truth, rules, persistence

This structure is scalable, testable, and debuggable.

---

A pure-logic extraction pattern is the practice of isolating business logic into side-effect-free functions (no UI, I/O, or framework calls) so they can be unit-tested deterministically. Client and server code become thin wrappers that call this logic, improving testability, reliability, and reuse.

Among experienced Anvil developers, pure-logic extraction is a common and recommended best practice, though not always followed by beginners. Anvil’s form-centric model encourages inline logic, but scalable, maintainable apps consistently separate pure logic for testing, reuse, and reliability across client–server boundaries.

Below is a clear, minimal, Gold-Standard example of the pure-logic extraction pattern for unit testing, shown the Anvil way.

The key idea:
👉 Business rules live in pure functions
👉 Client and Server only adapt inputs/outputs

1️⃣ Pure Logic Module (NO Anvil imports)
order_logic.py ← Pure logic
# Pure Python module (no anvil imports)


from typing import Dict




class OrderLogicError(Exception):
    """Raised when order calculation rules are violated."""




def calculate_order_total(
    price: float,
    tax_rate: float,
    max_total: float
) -> Dict[str, float]:
    """
    Pure business logic for calculating an order total.


    Args:
        price (float): Base price (>= 0).
        tax_rate (float): Tax rate (>= 0).
        max_total (float): Maximum allowed total.


    Returns:
        dict: subtotal, tax, total


    Raises:
        OrderLogicError: If inputs are invalid or rules violated.
    """


    if price < 0:
        raise OrderLogicError("Price must be non-negative")


    if tax_rate < 0:
        raise OrderLogicError("Tax rate must be non-negative")


    subtotal = price
    tax = subtotal * tax_rate
    total = subtotal + tax


    if total > max_total:
        raise OrderLogicError("Order total exceeds maximum allowed")


    return {
        "subtotal": subtotal,
        "tax": tax,
        "total": total
    }

✔ No UI
✔ No database
✔ No server calls
✔ Fully deterministic
✔ Easily unit-testable

2️⃣ Server Module (Thin Wrapper)
server_orders.py
import anvil.server
from anvil.tables import app_tables
import logging


from .order_logic import calculate_order_total, OrderLogicError
from .constants import MAX_ORDER_TOTAL, DEFAULT_TAX_RATE


logger = logging.getLogger(__name__)




@anvil.server.callable
def calculate_and_store_order(order_id: int, price: float):
    try:
        result = calculate_order_total(
            price=price,
            tax_rate=DEFAULT_TAX_RATE,
            max_total=MAX_ORDER_TOTAL
        )


        row = app_tables.orders.get(id=order_id)
        if not row:
            raise LookupError("Order not found")


        row.update(**result)
        return result


    except OrderLogicError as e:
        logger.info("Order validation failed", exc_info=True)
        raise


    except Exception:
        logger.critical("Unexpected server error", exc_info=True)
        raise

✔ Server enforces truth
✔ Logic already tested elsewhere
✔ No duplicated rules

Client Module (Thin Wrapper)
client_orders.py
import anvil.server




def submit_order(order_id: int, price_text: str):
    price = float(price_text)  # UI-level validation only
    return anvil.server.call(
        "calculate_and_store_order",
        order_id,
        price
    )

✔ Client coordinates
✔ No business rules
✔ UI stays clean

4️⃣ Unit Test (Pure Python)

This runs outside Anvil.

from order_logic import calculate_order_total, OrderLogicError




def test_valid_order():
    result = calculate_order_total(100, 0.2, 500)
    assert result["total"] == 120




def test_negative_price():
    try:
        calculate_order_total(-1, 0.2, 500)
        assert False
    except OrderLogicError:
        assert True

✔ Fast
✔ Deterministic
✔ No server required
✔ No UI required

Mental Model (Critical)
UI (Form)
   ↓
Client Module (coordination)
   ↓
Server Module (persistence + security)
   ↓
PURE LOGIC (rules, math, decisions)
Why this pattern is “Gold Standard”

Business rules exist once

Bugs are caught before deployment

Server and client stay thin

Refactoring is safe

Testing is trivial

Below is the correct, idiomatic way to test pure-logic extraction inside Anvil, without external tooling and without polluting production code.

This is how experienced Anvil developers do it.

Goal

Run tests inside the Anvil IDE

Test pure logic only

No UI

No Data Tables

No server calls

Fail loudly during development

1️⃣ Keep the Pure Logic Module (unchanged)

This stays exactly as before.

order_logic.py (Pure Logic)
from typing import Dict




class OrderLogicError(Exception):
    pass




def calculate_order_total(
    price: float,
    tax_rate: float,
    max_total: float
) -> Dict[str, float]:


    if price < 0:
        raise OrderLogicError("Price must be non-negative")


    if tax_rate < 0:
        raise OrderLogicError("Tax rate must be non-negative")


    subtotal = price
    tax = subtotal * tax_rate
    total = subtotal + tax


    if total > max_total:
        raise OrderLogicError("Order total exceeds maximum allowed")


    return {
        "subtotal": subtotal,
        "tax": tax,
        "total": total
    }

✔ No Anvil imports
✔ Testable anywhere

2️⃣ Create a Server Test Module

In Anvil:

Add a Server Module

Name it: test_order_logic

Do not decorate anything with @anvil.server.callable

This module exists only for development.

3️⃣ Write Tests Using Plain assert
test_order_logic.py
from .order_logic import calculate_order_total, OrderLogicError
from .constants import MAX_ORDER_TOTAL, DEFAULT_TAX_RATE




def test_valid_order():
    result = calculate_order_total(
        price=100.0,
        tax_rate=DEFAULT_TAX_RATE,
        max_total=MAX_ORDER_TOTAL
    )


    assert result["subtotal"] == 100.0
    assert result["tax"] == 20.0
    assert result["total"] == 120.0




def test_negative_price():
    try:
        calculate_order_total(
            price=-1,
            tax_rate=DEFAULT_TAX_RATE,
            max_total=MAX_ORDER_TOTAL
        )
        assert False, "Expected OrderLogicError"
    except OrderLogicError:
        pass




def test_excessive_total():
    try:
        calculate_order_total(
            price=MAX_ORDER_TOTAL,
            tax_rate=1.0,
            max_total=MAX_ORDER_TOTAL
        )
        assert False, "Expected OrderLogicError"
    except OrderLogicError:
        pass

✔ Fast
✔ Deterministic
✔ No Anvil runtime needed

4️⃣ Add a Test Runner Function (Manual Trigger)

Still inside the same server module:

def run_all_tests():
    """
    Run all pure-logic tests.
    Any assertion failure will raise immediately.
    """
    test_valid_order()
    test_negative_price()
    test_excessive_total()


    return "All tests passed"
5️⃣ How to Run Tests in Anvil (Very Important)
Option A — Run from the Server Console (BEST)

Open Server Modules

Open test_order_logic

Open the Python console

Run:

run_all_tests()

✅ If a test fails → traceback appears
✅ If all pass → "All tests passed"

Option B — Run Automatically in Development (Optional)

In a development-only server module:

if __name__ == "__main__":
    run_all_tests()

⚠️ Use this only in dev — remove before production.

6️⃣ What This Gives You (Why This Matters)

Tests run inside Anvil

No external test runner needed

Fail-fast with real tracebacks

Business rules verified before UI exists

Refactoring-safe logic

7️⃣ What You Should NOT Do

❌ Test through Forms
❌ Test through button clicks
❌ Test via server calls
❌ Mix tests with production server code

Mental Model (Lock This In)
Pure Logic  ← tested here, fast, isolated
   ↑
Server     ← thin wrapper
   ↑
Client     ← coordination
   ↑
UI         ← presentation only
Final Rule (Gold Standard)

If logic is important enough to exist, it is important enough to be tested —
and in Anvil, that means testing it in a server module with pure asserts.