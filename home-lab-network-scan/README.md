# [Project Name]

**Date(s):** [start – end]
**Status:** `Not Started` / `In Progress` / `Complete` / `Remediated` / `Monitoring`
**Environment:** [home lab / cloud sandbox / CTF / client name (redact if needed)]
**Tools used:** [e.g., Nmap, Wireshark, Metasploit, Burp Suite, Splunk]

---

## 1. Objective
*What were you trying to accomplish, and why does it matter?*

- Goal:
- Why this matters (business/security reason, not just "practice"):

---

## 2. Scope
*What was in bounds, what wasn't, and what assumptions did you make?*

- In scope:
- Out of scope:
- Assumptions / constraints:
- Authorization (if applicable — always note this for real environments):

---

## 3. Method
*What did you actually do, in order? This is your reasoning trail — write it as you go, not after.*

| Step | Action | Tool/Command | Result / Observation |
|------|--------|--------------|----------------------|
| 1 | | | |
| 2 | | | |
| 3 | | | |

> Tip: Note when something *didn't* go as expected and what you changed — that pivot is the most valuable part of this section for a portfolio reader.

---

## 4. Findings
*What did you discover? Be specific — vulnerabilities, misconfigurations, gaps.*

- Finding 1:
  - Severity: `Critical` / `High` / `Medium` / `Low` / `Informational`
  - Details:
- Finding 2:
  - Severity:
  - Details:

---

## 5. Impact / Risk
*So what? What could this have led to if left unaddressed?*

- Potential impact:
- Likelihood:
- Who/what is affected:

---

## 6. Remediation
*What did you fix, or what would you recommend, and how?*

| Finding | Recommended Fix | Status | Owner (if applicable) |
|---------|-----------------|--------|------------------------|
| | | | |

---

## 7. Evidence
*Logs, screenshots, configs — sanitize anything sensitive (real IPs, org names, credentials) before publishing.*

- [ ] Screenshot: before state
- [ ] Screenshot: after state / proof of exploit or fix
- [ ] Relevant log excerpts
- [ ] Config diffs

Store raw evidence in `/evidence` subfolder — reference it here, don't paste huge blobs inline.

---

## 8. Lessons Learned
*What would you do differently next time? What surprised you?*

-

---

## Appendix: Audience-specific cuts

**For portfolio (GitHub README / write-up):**
Lead with Objective + Impact. Keep the reasoning narrative in Method. Redact sensitive details. This is where you show judgment, not just tool output.

**For work report (status update / ticket):**
Lead with Findings + Impact/Risk. Cut narrative from Method down to a one-line summary. Add a clear Status and Next Action line at the top:
> **Status:** [x] | **Next action:** [x] | **Owner:** [x]
