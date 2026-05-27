# Bugula

Browser-based UI test automation tool. Record test sessions in a real browser, manage them in a local SQLite database, and replay them headlessly — from the GUI or via CLI.

## General

A testcase file consists of a `meta` block and a list of steps under `testcases`.

### YAML Structure

```yaml
meta:
  url: https://example.com
  browser: chrome
  username: user
  password: secret
  private: false

testcases:
  - url: https://example.com
    method: link
    description: Homepage reachable

  - method: form_input
    source_url: https://example.com/login
    selector: input[name="q"]
    input_value: hello
    submit_key: enter
    description: Submit search form
```

### Meta Fields

**url** — Required. Start URL the browser connects to first.
**browser** — Optional. `chrome` (default), `edge`, or `firefox`.
**username** — Optional. Login username passed to the login handler.
**password** — Optional. Login password passed to the login handler.
**private** — Optional. `true` to open the browser in private / incognito mode.

## Methods & Fields

Every step shares these common fields:

**method** — Required. Name of the method (see below).
**description** — Required. Label shown in test results.
**url** — Target URL. Required for `link`; informational for other methods.
**source_url** — Page to open before searching for an element.
**selector** — CSS selector used to locate an element on the page.
**element_text** — Visible text of the element to find and click.
**input_value** — Text to type (`form_input`) or seconds to wait (`wait`).
**submit_key** — Key pressed after typing. Accepted values:

```
enter / return   ↵  Confirm / submit
tab              →  Move focus to next field
escape               Close / cancel
```

**assert_text** — Text that must appear on the page (used by `assert_text` method).

---

### link

Navigates directly to `url` and verifies the page loads correctly. Fails on HTTP error titles (404, 500, 403 …), empty pages, or when the browser is redirected to a different host — for example an OAuth provider at `localhost`.

**url** — Required. Destination URL.

---

### nav_click

Opens `source_url`, searches navigation elements (nav, sidebar, header …) by `element_text`, and clicks the match.

**source_url** — Page that contains the navigation element.
**element_text** — Visible link or button text.

---

### click

Clicks any element identified by a CSS selector. Optionally opens `source_url` first.

**selector** — Required. CSS selector of the target element.
**source_url** — Page to open first (optional).

---

### form_input

Finds an input field by `selector`, clears it, types `input_value`, and optionally presses `submit_key`.

**selector** — Required. CSS selector of the input field.
**input_value** — Text to type into the field.
**submit_key** — Key to press after typing (see Common Fields above).
**source_url** — Page to open first (optional).

---

### assert_text

Checks that `assert_text` appears somewhere on the page. Fails if the text is not found.

**assert_text** — Required. Text that must be present.
**selector** — Optional. Limit the search to this element.
**source_url** — Page to open first (optional).

---

### modal

Clicks a modal-trigger element (`data-toggle="modal"`, `aria-haspopup` …) by `element_text` and verifies that a dialog appears.

**source_url** — Page containing the trigger element.
**element_text** — Visible text of the trigger.

---

### tab

Clicks a tab element (`[role="tab"]`) by `element_text` and checks that the DOM changes.

**source_url** — Page containing the tab.
**element_text** — Visible text of the tab.

---

### pagination

Clicks a pagination button identified by `element_text` or `aria-label`.

**source_url** — Page containing the pagination control.
**element_text** — Button text or `aria-label` (e.g. `Next`).

---

### table_row

Opens `source_url` and clicks the first visible data row of a table. Checks whether the URL or DOM changes afterwards.

**source_url** — Required. Page containing the table.

---

### wait

Pauses execution for a fixed number of seconds. Useful after animations or async loading.

**input_value** — Seconds to wait (default: `1.0`).

## Examples

### Reachability Check

Verify that a URL loads and is not redirected to another host.

```yaml
meta:
  url: https://example.com

testcases:
  - url: https://example.com
    method: link
    description: Homepage reachable
```

### Login Flow

Enter credentials, submit the form, and assert the dashboard loads.

```yaml
meta:
  url: https://example.com
  browser: chrome

testcases:
  - url: https://example.com
    method: link
    description: Homepage reachable

  - method: form_input
    source_url: https://example.com/login
    selector: input[name="username"]
    input_value: admin
    description: Enter username

  - method: form_input
    source_url: https://example.com/login
    selector: input[name="password"]
    input_value: secret
    submit_key: enter
    description: Enter password and submit

  - method: assert_text
    source_url: https://example.com/dashboard
    assert_text: Dashboard
    description: Dashboard loaded
```

### Navigation & Table

Click a nav link, open a table page, and click the first row.

```yaml
testcases:
  - method: nav_click
    source_url: https://example.com/dashboard
    element_text: Reports
    description: Navigate to Reports

  - method: table_row
    source_url: https://example.com/reports
    description: Open first report row

  - method: assert_text
    assert_text: Report Details
    description: Detail page loaded
```

### Search with Wait

Type a search query, wait for async results, then assert the content.

```yaml
testcases:
  - method: form_input
    source_url: https://example.com
    selector: input[name="q"]
    input_value: annual report
    submit_key: enter
    description: Search for annual report

  - method: wait
    input_value: "2"
    description: Wait for results to load

  - method: assert_text
    assert_text: annual report
    description: Search results contain query
```

## Setup

### Requirements

- Python 3.11+
- Google Chrome, Microsoft Edge, or Firefox
- Matching ChromeDriver / EdgeDriver / GeckoDriver on `PATH`

Install dependencies:

```
pip install -r requirements.txt
```

### Running the GUI

```
python main.py
```

## Automated CLI

```
python main.py --automated
```

Runs all testcases marked as **Automated** in the Testing view headlessly (no browser window).