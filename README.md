# QA & Testing Knowledge Portal for Oracle Fusion Cloud ERP programs

A reusable, single-file reference portal for QA and testing on an Oracle Fusion Cloud ERP implementation. It ships with fictional sample content for a program called **Lime**. Every program-specific value lives in `portal-config.json`, so reusing it on another program means editing data, not code.

## Files

| File | Purpose |
|---|---|
| `qa-testing-portal.html` | The complete portal: HTML, CSS and JavaScript in one file, with the configuration embedded. |
| `portal-config.json` | The same configuration as a standalone file. Edit this one. |
| `sample-imports/` | Fictional CSV and Excel files that exercise the Jira, ALM and Excel import. |

## Technology choice

**Plain HTML, CSS and JavaScript in one file, not React with Material-UI.** Reasons: it opens by double-click with no build step or server; it can be dropped on SharePoint or any static host; it has no dependency tree to patch; and the configuration is cleanly separated from rendering. Two public libraries load from cdnjs.cloudflare.com: Chart.js (charts) and SheetJS (Excel import). If they cannot load, the portal still works: charts are replaced by their data tables and Excel files can be saved as CSV. Open Sans loads from Google Fonts and falls back to Arial.

## Opening it

1. Double-click `qa-testing-portal.html`. It uses the configuration embedded in the file.
2. Or host `qa-testing-portal.html` and `portal-config.json` in the same folder on a web server. The portal then loads `portal-config.json` from that folder, so you can change the configuration without touching the HTML.
3. In the portal you can also use **Data sources and integration > Configuration** to download the current configuration or load a different `portal-config.json` for the session.

Nothing is stored in the browser (no localStorage or sessionStorage). Imported data lives in memory until the page is reloaded. If the portal will ever hold real program data, host it in an access-controlled location.

## Site structure

**Start**

- Home: See where testing stands on Lime

**Plan**

- Test strategy: Plan the testing approach
- Test cycles: Know the rules of each test cycle
- Entry and exit criteria: Check entry and exit criteria before each cycle
- Scope and coverage: Find what is in scope and who covers it

**Prepare**

- RICE objects: Track reports, interfaces, conversions and extensions
- Data conversion: Confirm data conversion readiness
- Security and access: Confirm security and access readiness

**Execute**

- Tester guide: Run scripts and record results
- Business user UAT guide: Complete UAT and give your sign-off
- Defect management: Log, triage and resolve defects
- Traceability: Trace every story to its defects

**Measure**

- Metrics and KPIs: Read and calculate the KPIs correctly
- Semaphore (RAG) rules: Apply Green, Amber and Red consistently
- Data sources and integration: Connect Jira, ALM and Excel data to the dashboard
- Data quality: Find and fix data quality issues

**Reference**

- Roles and responsibilities: Know who does what
- Templates and checklists: Copy templates and checklists
- Glossary and FAQ: Look up terms and common questions
- Assumptions and open items: Review assumptions and open items

Always available: global search across all pages, a left navigation with a role-based **Start here** path, a data banner that shows whether sample or imported data is in use, and copy and print for templates and checklists.

## Loading data from Jira, ALM or Excel

1. Open **Data sources and integration > Import data**.
2. Choose where the file comes from and what it contains (test results, defects or user stories).
3. Choose a CSV, TSV or Excel file. For Excel, pick the sheet.
4. Check the column mapping. Columns are matched by name where possible. Required fields must be mapped.
5. Read the issue list (unrecognized statuses, unreadable dates, rows without an ID), then select **Apply**.

Values that match no known alias are left unmapped and reported. They are never guessed and never counted as Passed or as zero. Add aliases under `enums` in the configuration. Direct connections to Jira or ALM are not included: ALM-specific details are marked **ALM adapter required**.

Try it with the files in `sample-imports/`. `lime_messy_test_results.csv` is built to show the issue reporting.

## Changing the configuration

All keys below are in `portal-config.json`. Use `null` for any value that is not known yet: the page then shows a visible "To be confirmed by project team" placeholder.

**Change the client or program.** Edit `program.name`, `program.clientName`, `program.overview`. The token `{{program}}` inside any text is replaced by `program.name`.

**Set the European tax country, dashboard URL or defect source.** Edit `program.euTaxCountry`, `program.dashboard.url` and `program.defectSource.selected` (`ALM`, `Jira` or `Both`).

**Add a module.** Add an object to `modules`: `{ "id": "ap", "name": "Payables", "type": "Oracle module", "enabled": true }`. To place it in the matrix, add `"ap": "primary"` or `"supporting"` to the `modules` map of one or more entries in `processes`. Set `enabled` to `false` to exclude a module while keeping it visible as Excluded.

**Add a process.** Add an object to `processes`: `{ "id": "P11", "name": "...", "description": "...", "modules": { "ar": "primary" } }`.

**Add a cycle.** Add an object to `cycles` (`id`, `short`, `name`, `enabled`, `summary`, `facts`). Then add its criteria under `criteria.<id>` (`entry` and `exit` lists), add its spellings under `enums.cycle.<id>`, and add its `id` to the `cycles` list of each KPI in `kpis` that applies to it. **Regression** is already defined and set to `enabled: false`, so it shows "Not defined in current scope". Set it to `true` and fill in `facts` and `criteria.regression` to activate it.

**Add or change a threshold.** Entry and exit limits are in `thresholds.criteria` (`value`). RAG ratings are in `thresholds.rag`: set `green` and `amber` and choose `direction` (`higherIsBetter` or `lowerIsBetter`). Remove `sample` once the value is confirmed. To tie an entry or exit criterion to a limit, set its `thresholdKey`.

**Add a KPI.** Add an object to `kpis` with `inputs` and `outputs` (`num` and `den` list input keys; `scale`; `unit`; `capped`; `thresholdKey`). The calculator, catalog and rating pick it up automatically.

**Add RICE objects, conversion objects, defect severities, templates, glossary terms.** Add to `rice.objects`, `conversion.objects`, `defects.severities`, `templates` and `glossary`. A conversion step counts as complete only when it has `evidence`.

**Replace the sample data.** Delete or empty the `sampleData` block. Pages then show Not Available until data is imported.

## Assumptions

Each assumption is also labeled on the page where it applies, and listed on the **Assumptions and open items** page.

- **A-01** The portal is a reference site. It reads exported files only and makes no live connection to Jira, ALM or any other system.
- **A-02** All sample content for the program called Lime is fictional and generic: processes, RICE object names, module links, criteria owners, RACI letters, glossary text and sample records.
- **A-03** Sample RAG thresholds exist for three metrics only (Execution %, Pass rate, average defect age) and are illustrative. Aging bucket boundaries are illustrative. All other thresholds, limits and SLAs are not set.
- **A-04** Execution % uses Passed + Failed + Blocked as the numerator and all planned, applicable scripts (Not Applicable excluded) as the denominator.
- **A-05** Defect Leakage is calculated for the receiving cycle: defects found in the cycle and attributed to an earlier cycle, divided by all defects found in the cycle.
- **A-06** Open defects are those with status Open, In Progress or Reopened. Resolved and Closed are not counted as open.
- **A-07** Status, severity, priority and cycle aliases (for example To Do mapped to Open) are generic and editable. Values that match no alias are flagged as unmapped, never guessed.
- **A-08** Sheet names and purposes for Testing_Daily_Status.xlsx are placeholders and must be confirmed against the real workbook.
- **A-09** The defect lifecycle transitions shown are generic and must be confirmed against the workflow in the defect source system.
- **A-10** Only the RICE complexity distribution (3 Simple, 6 Medium, 4 Complex, 7 Very Complex) comes from the scope. Object names, types and module links are illustrative.
- **A-11** 'Clientes' in the conversion object list is treated as Customers and covered by Customer Master.
- **A-12** Vertex integration is treated as an integration / boundary application. How it is tested in each cycle is to be confirmed.
- **A-13** Severity and priority definitions and the severity/priority matrix are generic industry conventions, not Oracle- or client-specific rules.
- **A-14** First-Pass Yield and Re-test Pass Rate need an attempt number per result. Without it, they show Not Available.
- **A-15** ALM-specific connection details are not provided. Exports from ALM can be loaded as CSV or Excel files. A direct feed needs an ALM adapter.
- **A-16** Each test and defect row is assigned to a cycle through the cycle aliases. Rows that match no alias are reported as unmapped and are not merged into any cycle.
- **A-17** The portal is one HTML file that loads Chart.js and SheetJS from a public CDN. Open it with internet access. Open Sans loads from Google Fonts and falls back to Arial.
- **A-18** Imported data is held in memory only. Reloading the page returns to the configuration and sample data.
- **A-19** Imported dates are read in ISO format and common day-month-year formats. Unreadable dates are flagged and are not guessed.
- **A-20** Coverage % counts all loaded stories as in scope because no out-of-scope flag exists in the sample dictionary.
- **A-21** Entry and exit criteria owners and evidence names are illustrative. Thresholds are not set.
- **A-22** The escalation path levels and the roles in the RACI table are illustrative. Triggers are not set.
- **A-23** Fixed record counts for Customer Master and Product Master, conversion evidence references, and conversion step statuses are sample values.

## Information still needed from the project team

| ID | Item | Suggested owner |
|---|---|---|
| O-01 | Confirm the client and program name, and the go-live date. | Program manager |
| O-02 | Confirm the final module list, exclusions and process-to-module mapping. | QA lead / Solution architect |
| O-03 | Confirm the European tax country for Cloud Tax. | Program manager |
| O-04 | Confirm how Vertex integration is tested in SIT and UAT. | Technical owner |
| O-05 | Provide entry and exit thresholds, including the SIT data completeness threshold. | QA lead |
| O-06 | Provide RAG thresholds for every KPI. | Program manager |
| O-07 | Provide defect response and resolution SLAs by severity. | QA lead |
| O-08 | Confirm the defect source of truth: Jira, ALM or both. | QA lead |
| O-09 | Confirm ALM version, export method and whether an adapter is planned. | Technical owner |
| O-10 | Provide Jira project keys, workflow status names and custom field names. | QA lead |
| O-11 | Provide the real layout of Testing_Daily_Status.xlsx. | QA lead |
| O-12 | Provide the testing dashboard URL. | Program manager |
| O-13 | Confirm the RICE object list, owners and complexity ratings. | Technical owner |
| O-14 | Confirm the conversion object list, expected record counts and the evidence repository. | Data lead |
| O-15 | Confirm security design status, SoD approach and the pod refresh schedule. | Technical owner |
| O-16 | Confirm the SIT business stakeholders and the UAT approvers. | Program manager / Business lead |
| O-17 | Decide whether a regression cycle is needed and, if so, define it. | Program manager |
| O-18 | Confirm the escalation path, triggers and the RACI. | Program manager |

## Quick test checklist

1. Navigation: open every page from the left menu. The page title changes and focus moves to the main heading.
2. Role path: choose each role in Start here. A numbered path appears and the links open the right pages.
3. Search: type 'denominator'. Results appear. Use the arrow keys and Enter to open one. Press Escape to close the list.
4. Filters: on Scope, RICE and Defects, change each filter. The row counts update and Clear filters restores them.
5. Calculator: on Metrics, calculate Pass rate with Passed 25, Failed 3, Blocked 2, Not Run 10. Expect 83.3% with denominator 30.
6. Missing data: clear one input in the calculator. Expect Not Available with a reason, not 0.
7. Zero denominator: enter 0 for Passed, Failed and Blocked. Expect Not Available.
8. Import: load a file from the sample-imports folder. The data banner changes and the pages update.
9. Modal: open a KPI or template. Press Escape. The dialog closes and focus returns to the button.
10. Copy and print: copy a template and paste it elsewhere. Print a template and the UAT checklist.
11. Keyboard only: tab through a page. Focus is always visible and nothing is skipped or trapped.
12. Contrast and zoom: zoom to 200% and check the layout. Check that status chips show text as well as color.
13. Storage: confirm in browser tools that localStorage and sessionStorage are empty.

This checklist is also on the **Assumptions and open items** page.

## Rules the portal enforces

- FUT/TUT, SIT and UAT are always reported separately and are never merged.
- Result rates use Passed + Failed + Blocked as the denominator, and the denominator is always shown next to the rate.
- Not Run, In Progress and Not Applicable are excluded from result rates. Not Run and Not Applicable are never counted as Passed.
- Missing data is never treated as zero. A metric that cannot be calculated shows Not Available and the reason.
- Thresholds, the European tax country and the defect source are read from the configuration. None is hardcoded.
- Security and access are readiness dependencies, not test results, unless an explicit test record exists.
- No credentials, tokens, API endpoints or real customer or employee data are included.
