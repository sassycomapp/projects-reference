In Anvil, **Slidesheets** are designed for situations where you want to combine the **structured nature of a spreadsheet** with the **guided, step-by-step experience of a presentation or form**.

A **typical use case** is when you need non-technical users to **enter, review, or update data in a controlled and intuitive way**, without exposing them to a raw database table or a complex multi-page app.


VALID - FULLY APPLICABLE with M3 Theme + Dependency.

CRITICAL INFO (Must Read)

text
**Slidesheets = M3 NATIVE pattern using NavigationRailLayout**

**Perfect for Mybizz guided workflows:**
- Customer onboarding → 3-slide flow  
- Record review → One slide per record
- Survey collection → Step-by-step

**Implementation (100% M3):**
NavigationRailLayout.show_sidesheet = True  # Entry 11
content_panel.clear() → add(Slide1Form())
Next button → content_panel.add(Slide2Form())
Data binds to self.item via Entries 64-65
text
# Slide navigation pattern:
@handle("m3_filled_button_next", "click")
def next_slide(self, **event_args):
    self.content_panel.clear()
    self.content_panel.add(Slide2(item=self.item))  # Data persists
Status: HIGHLY RECOMMENDED for Mybizz guided data entry/review workflows. Pure M3, zero extras needed.

---

## Typical Use Case: Guided Data Entry & Review

### Scenario

Imagine you are building an internal app for:

* collecting survey responses
* onboarding new customers
* reviewing applications (grants, loans, job candidates)
* updating records that normally live in a spreadsheet

Traditionally, people might:

* fill in a Google Sheet, or
* receive a long, overwhelming form

With **Slidesheets**, you can instead guide them through the data **one slide at a time**.

---

## How Slidesheets Help in This Scenario

### 1. One Record, One “Slide”

Each slide typically represents:

* one row in a Data Table, or
* one logical step in a workflow

For example:

* Slide 1: Basic information
* Slide 2: Details and selections
* Slide 3: Review and confirmation

This keeps users focused and reduces mistakes.

---

### 2. Spreadsheet-Like Structure (Without the Mess)

Behind the scenes:

* Data is still tabular (rows and columns)
* Easy to map directly to Anvil Data Tables

But for the user:

* They don’t see rows, formulas, or columns
* They see clear questions, fields, and instructions

This is ideal when users are uncomfortable with spreadsheets or shouldn’t edit everything freely.

---

### 3. Great for Review & Approval Workflows

Another very common use case is **reviewing records**:

Example:

* A manager reviews expense claims
* Each slide shows one claim
* Buttons like **Approve**, **Reject**, **Next**

This is far easier than scrolling through a large table.

---

### 4. Works Well on Tablets and Touch Devices

Because Slidesheets are:

* linear
* visual
* button-driven

They’re often used in:

* field data collection
* kiosks
* internal tools used on tablets

---

## Why Use Slidesheets Instead of a Normal Form?

Use Slidesheets when you want:

* A **guided, step-by-step flow**
* Fewer user errors
* A cleaner experience than a spreadsheet
* A UI that matches how people *think* through data, not how databases store it

---

### In short:

**Slidesheets are ideal when you want spreadsheet-backed data, but a presentation-style user experience.**

If you want, I can also:

* compare Slidesheets vs standard Forms
* show a concrete example tied to a Data Table
* explain how navigation and events usually work in Slidesheets

Just let me know 👍
