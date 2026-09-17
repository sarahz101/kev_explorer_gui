# KEV Explorer

A desktop application for browsing the CISA Known Exploited Vulnerabilities catalogue — the authoritative list of software flaws confirmed to have been exploited by attackers in the wild.

Built with Python and PySide6 (Qt). Search 1,713 vulnerability records by plain text or regular expression, filter by vendor, year and ransomware association, and view summary statistics that recalculate against whatever you have filtered.

---

## Features

**Live search, two modes.** Results filter as you type. Plain-text mode escapes the query so `v1.2` matches a literal dot rather than any character. Regular expression mode passes the query through untouched, so `buffer (over|under)flow` matches either phrasing and `CVE-202[45]-` matches two years at once. An invalid pattern shows an inline error and keeps the previous results on screen, rather than clearing the table while you are mid-pattern.

**Composable filters.** Vendor, year added, and ransomware association combine with the search box. All 283 vendors are read from the data rather than hardcoded.

**Detail view.** Selecting a row shows the full description, the required remediation action, weakness types (CWE), version numbers mentioned in the description, and clickable reference links. Where a record cross-references another CVE — an exploit chain, where two flaws are combined in a single attack — those are surfaced too.

**Statistics that follow your filters.** Top vendors, most common weakness types, and additions per year, each with an inline bar chart. Filter to one vendor and the statistics describe that vendor alone.

**Derived metrics.** The remediation window — days between a vulnerability entering the catalogue and its remediation deadline — is computed from two date fields and appears nowhere in the source data. It averages 42.4 days across the catalogue.

---

## Running it

Requires Python 3.9 or newer.

```bash
pip install PySide6
python kev_explorer.py
```

`kev.csv` must sit in the same folder as the script — it is loaded by relative path.

To refresh the dataset:

```bash
curl -L -o kev.csv https://www.cisa.gov/sites/default/files/csv/known_exploited_vulnerabilities.csv
```

---

## How it is built

The program is split into a back end and a front end, and the boundary is strict.

The **back end** is plain Python — loading, validating, filtering, searching, sorting and summarising. It imports nothing from Qt and can be tested without opening a window. The **front end** builds the interface and calls back-end functions in response to what the user does; it contains no filtering logic and no regular expressions of its own.

A consequence worth naming: porting this to a command-line tool or a web interface would mean rewriting the front end and reusing the back end unchanged.

### Regular expressions

Six compiled patterns, each doing a different job:

| Pattern | Purpose |
|---|---|
| `^CVE-\d{4}-\d{4,7}$` | Validating a complete CVE identifier |
| `CVE-\d{4}-\d{4,7}` | Finding identifiers inside prose (same pattern, unanchored) |
| `CWE-\d+` | Extracting weakness identifiers |
| `https?://[^\s;,]+` | Extracting reference URLs |
| `^(\d{4})-(\d{2})-(\d{2})$` | Validating and parsing ISO dates |
| `\b\d+\.\d+(?:\.\d+)*\b` | Extracting version numbers |

The first two are identical apart from the anchors, which is the whole difference between *is this string an identifier* and *does this text contain one*.

The sequence number is `{4,7}` rather than `{4}` because CVE numbers were capped at four digits until the 2014 format change. A stricter pattern would silently drop `CVE-2021-44228` — Log4Shell.

Date handling uses two layers. The regex validates shape; a `try`/`except` catches dates that are well-formed but impossible, such as `2024-02-31`. A pattern can check format but not meaning.

---

## Data

Source: [CISA Known Exploited Vulnerabilities Catalog](https://www.cisa.gov/known-exploited-vulnerabilities-catalog)

A work of the United States federal government, published for free public use. CISA maintains it as the authoritative record of vulnerabilities with confirmed exploitation in the wild, intended as an input to vulnerability-management prioritisation.

The snapshot included here has 1,713 records spanning 2021–2026. The catalogue is updated regularly; re-download to refresh.

---

## Notes

Written as a university assignment. Standard library plus PySide6 only — no pandas, no matplotlib. The bar charts are drawn with Unicode block characters, which keeps the dependency list to one.

## Licence

Code released under the MIT Licence. The KEV catalogue itself is US government work in the public domain.
