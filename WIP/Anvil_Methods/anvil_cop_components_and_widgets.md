In **Anvil**, the answer is not “one is always better” — it depends on **scope, reuse, and encapsulation**.
That said, there **is a clear preference in most real apps**.

---

## Short answer

> **Prefer a Custom Component.**
> Use a **Widget only when you are wrapping external JS/CSS or browser-level behavior.**

---

## What each one really is

### **Custom Component**

A **Custom Component** is:

* A reusable **Anvil Form**
* Written entirely in **Python**
* Uses standard Anvil components
* Fully supported by:

  * Data Bindings
  * Events
  * Properties
  * Design-time configuration

Think of it as:

> “A reusable Form with a clean public API”

---

### **Widget**

A **Widget** is:

* A wrapper around **HTML / CSS / JavaScript**
* Uses the **HTML panel**
* Often relies on:

  * JS libraries
  * DOM manipulation
  * Browser APIs

Think of it as:

> “Escaping Anvil to talk directly to the browser”

---

## Comparison (practical, Anvil-focused)

| Criteria              | Custom Component | Widget                |
| --------------------- | ---------------- | --------------------- |
| Python-only           | ✅ Yes            | ❌ No                  |
| Uses Anvil components | ✅ Yes            | ❌ No                  |
| Design-time friendly  | ✅ Yes            | ❌ Limited             |
| Data bindings         | ✅ Yes            | ❌ No                  |
| Event handling        | ✅ Native         | ⚠️ Manual JS → Python |
| Maintainability       | ✅ High           | ❌ Lower               |
| Debugging             | ✅ Easy           | ❌ Harder              |
| External JS libraries | ❌ No             | ✅ Yes                 |
| DOM-level control     | ❌ No             | ✅ Yes                 |

---

## When a **Custom Component** is preferable (most cases)

Use a **Custom Component** when:

* You want **reuse**
* You want **clean separation of UI logic**
* You want **Anvil-native behavior**
* You are composing:

  * Labels
  * Buttons
  * Repeaters
  * Data-bound components
* You want other developers to easily use it

**This is the default choice in well-structured Anvil apps.**

---

## When a **Widget** is preferable (special cases)

Use a **Widget** only when you **must**:

* Integrate a **third-party JS library** (charts, maps, editors)
* Access **browser APIs** not exposed by Anvil
* Do custom **CSS-heavy or DOM-level rendering**
* Embed complex HTML content

Typical valid examples:

* Leaflet / Mapbox maps
* Monaco / CodeMirror editors
* Custom canvas rendering
* Advanced drag-and-drop frameworks

---

## Rule of thumb (important)

> **If it can be built as a Custom Component, it should be.**
> Reach for a Widget **only when Anvil’s component model cannot do the job**.

---

## Architectural best practice

A very common and **clean** pattern:

* **Custom Component**
  → handles layout, data, events, API
* **Widget (internally)**
  → only if needed, hidden behind the component interface

This keeps the rest of your app **pure Anvil**.

---

### Bottom line

* **90% of cases** → **Custom Component**
* **10% of cases** → **Widget**
* **0% of cases** → Widget “just because”

