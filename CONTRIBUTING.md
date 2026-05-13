# Contributing

Contributions that keep the checklist accurate and current are very welcome.

---

## What makes a good contribution

**Good fits:**
- A vulnerability pattern that AI coding assistants introduce consistently and is not yet in the list
- A better detection method for an existing item (grep pattern, semgrep rule, AST query)
- A severity correction with a clear justification
- A new AI-specific category, such as multi-agent trust boundaries or model context protocol misuse
- Fixing an inaccuracy, duplicate, or item that no longer applies

**Not a good fit:**
- Generic security advice without a detection method attached
- Highly platform-specific items that do not generalize beyond one cloud provider or framework
- Items that only apply to a single library with no broader pattern

---

## Schema for `vulnerabilities.md`

Every item follows this format:

```markdown
| {N}.{n} | {Vulnerability name} | {Detection} | {Severity} |
```

**Fields:**

| Field | Values | Notes |
|-------|--------|-------|
| `N.n` | e.g. `6.26` | Category number + sequential item number |
| Vulnerability name | Plain text | Describe what is wrong, not just what is missing |
| Detection | `[S]`, `[R]`, `[C]`, or a combination | See the legend below |
| Severity | 🔴 🟠 🟡 🟢 | See the severity guide below |

**Detection codes:**
- `[S]` Static analysis: findable via grep, AST scan, or reading the code
- `[R]` Runtime: requires a running server to detect (HTTP headers, timing, behavior under test)
- `[C]` Config or infrastructure: requires inspecting environment settings, cloud config, or provider dashboard

**Severity guide:**

| Emoji | Level | Criteria |
|-------|-------|----------|
| 🔴 | Critical | Exploitable without authentication, or leads directly to RCE, data loss, or account takeover |
| 🟠 | High | Significant risk that requires some context, chaining, or user interaction to exploit |
| 🟡 | Medium | Non-trivial to exploit but meaningful impact if it is exploited |
| 🟢 | Low | Defense in depth, best practice, low exploitability |

---

## Schema for `prompt.md`

Each check in `prompt.md` maps one-to-one to an item in `vulnerabilities.md` by rule ID (e.g. `6.26`).

Format for a new check:

```markdown
**{N}.{n}** Search for {specific code pattern or construct}. {What to look for}. Flag if {condition}.
```

Rules:
- Be specific about what to search for. Write "Search for `dangerouslySetInnerHTML`" not "Look for XSS problems."
- State the exact condition that triggers a flag.
- If a check cannot be automated by code search, mark it `[C]` and describe what the reviewer should verify in their provider dashboard or environment.

---

## Keeping the summary table current

When you add items, update the summary table at the bottom of `vulnerabilities.md`:

```markdown
| {Category} | {total items} | {critical count} | {high count} | {medium count} | {low count} |
```

Update the Total row too.

---

## Adding a new category

If a new category is warranted rather than adding items to an existing one:

1. Add it to `vulnerabilities.md` with a new `## {N}. {Category Name}` heading
2. Add the corresponding section to `prompt.md` using `### Category {N} - {Category Name}`
3. Add a row to the summary table in `vulnerabilities.md`
4. Add a row to the category table in `README.md`
5. Note the new category in `CHANGELOG.md` under `[Unreleased]`

---

## Pull request checklist

Before submitting:

- [ ] New items follow the schema above exactly, with the correct column order
- [ ] Rule IDs are sequential and do not collide with existing ones
- [ ] `prompt.md` has a matching check for every new item in `vulnerabilities.md`
- [ ] The summary table at the bottom of `vulnerabilities.md` is updated
- [ ] `CHANGELOG.md` has an entry under `[Unreleased]`
- [ ] The PR description explains: what the vulnerability is, why AI tools introduce it, and how it was observed in the wild

---

## Local development

No build step needed. Everything is Markdown.

```bash
git clone https://github.com/a-leks/genai-app-security-checklist
cd genai-app-security-checklist
# Edit vulnerabilities.md and prompt.md
# Preview in any Markdown renderer
```

To test the prompt end-to-end:

```bash
cd /your/test/project
claude "$(cat /path/to/genai-app-security-checklist/prompt.md)"
```

---

## Code of conduct

Keep discussions technical and factual. Security topics involve dual-use material, so contributions should stay focused on detection and remediation rather than exploitation techniques.
