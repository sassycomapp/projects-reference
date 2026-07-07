# PDLF Standards Library — Testing Methodology

**Location:** `C:\pdlf\standards-library\testing-methodology-standards.md`
**Scope:** Project-agnostic testing method. Any project's specific test *plan* (which flows,
which roles) is that project's own document (see `pdlf-testing-policy.md` Section 6 for the
shape such a plan should take).
**Source:** Extracted from `mb-3-cs` (testing-standards.md, test-specifications.md), generalized.
**Relationship to `pdlf-testing-policy.md`:** that file is PDLF's own process reference, written
for the build loop (Step 16). This file is the underlying standard it's built on — kept separate
so the standard itself is reusable outside PDLF's specific step numbering too.

---

## 1. Testing Levels

| Level | Scope | Runtime | Risk | Rule |
|---|---|---|---|---|
| 1 | Pure logic | Local Python | Zero | Continuous — run on every change |
| 2 | Integration via Uplink | Local + live Anvil | Medium | Only after Level 1 passes and a backup exists |
| 3 | End-to-end manual verification | Browser | Low | After a full work increment is complete |

## 2. Pure Function Characteristics (Level 1)

- No `import anvil` statements.
- No Data Tables access, no server function calls.
- No side effects.
- Deterministic.
- Scope: calculations, validations, transformations — not orchestration.

## 3. Test Structure Standard

```python
def test_feature_scenario():
    """Test: Clear description"""
    # Arrange
    input_value = ...
    expected_output = ...
    # Act
    result = function_under_test(input_value)
    # Assert
    assert result == expected_output
```

- Naming: `test_[feature]_[scenario]`, snake_case, grouped by prefix.
- Assertions: plain `assert`.
- A `run_all_tests()`-style runner function per module.
- A failed test stops progress and gets fixed immediately — it is not logged and deferred.

## 4. Integration Testing Safety Protocol (Level 2)

**Before:** all pure logic tests pass; a backup exists; the Uplink connection is verified.

**During:** read before writing; mark any test data with `source='Test'`; verify each write
after it happens, don't assume it succeeded.

**After:** clean up test data; verify the cleanup actually succeeded; create a second backup on
success.

**Two backup points, not one** — a defensive backup before integration code touches the live
app, and a checkpoint backup once Level 2 passes. This is intentional, not redundant: the first
backup protects against the integration attempt itself.

## 5. Common Test Patterns

**Happy path:**
```python
def test_function_valid_input():
    result = function(valid_input)
    assert result == expected_output
```

**Error handling:**
```python
def test_function_invalid_input():
    try:
        function(invalid_input)
        assert False, "Expected exception"
    except ExpectedException as e:
        assert "expected message" in str(e).lower()
```

**Boundary conditions:** test zero, minimum, and maximum values explicitly — not just "typical"
inputs.

## 6. Test Plan Shape (for any project's own test specification)

A concrete test plan, regardless of project, should include:
- **E2E happy-path specs** per critical flow: preconditions, a numbered steps table (action →
  expected result), a performance gate where relevant, explicit data-validation checks.
- **Edge cases**, listed separately from the happy path.
- **Error-path test cases**, one per named failure scenario, each with its own specific expected
  behavior.
- **An RBAC/access coverage matrix**, if the project has roles: a form/tab access matrix, a test
  procedure (log in as role → attempt access → verify granted/denied → verify the nav link is
  hidden, not merely disabled), and a mechanical check — every server function should be grepped
  for its required auth decorator; a function without one is a bug, not a style note.

A real test plan is typically produced as a direct response to specific review findings ("no
error-path tests," "no RBAC matrix") rather than written speculatively — the review names the
gap, the test plan closes it.

## 7. RBAC Decorator Pattern

```python
@authenticated_endpoint(roles=['Owner', 'Manager', 'Admin'])
def get_contacts():
    ...
```

Every server function that isn't intentionally public should carry an equivalent decorator. The
mechanical test: grep all server modules for the decorator; any function without one, that isn't
explicitly documented as intentionally public, is a finding.

---

*Testing Methodology Standards v1.0. The levels and safety protocol are the reusable part — a
project's specific flows, roles, and data belong in that project's own test specification.*
