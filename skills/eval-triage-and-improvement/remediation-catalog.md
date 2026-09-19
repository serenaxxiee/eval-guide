### Step 6: Map to Remediation

For detailed remediation steps by Step 7 root bucket, operational subtype, and eval-set failure pattern, read the playbook files:
- **Full triage decision tree**: Read `triage-and-improvement-playbook/triage-decision-tree.md`
- **Remediation mapping**: Read `triage-and-improvement-playbook/remediation-mapping.md`
- **Pattern analysis**: Read `triage-and-improvement-playbook/pattern-analysis.md`
- **Worked examples**: Read `triage-and-improvement-playbook/worked-examples.md`

#### Quick Remediation Reference

**Eval-setup fixes:**

| Sub-Type | Fix |
|----------|-----|
| Outdated expected answer | Update expected value to match current source content |
| Overly rigid grader | Switch to Compare Meaning, or broaden keyword set |
| Unrealistic test case | Rewrite input using actual user language |
| Wrong eval method | Change method to match the eval-set purpose and evidence type |
| Grader error/bias | Review rubric, add examples, consider deterministic method |

**Agent-quality fixes — agent configuration / knowledge / tools:**

| Failure pattern | Common Fix |
|-----------------|-----------|
| Factual accuracy (wrong source) | Review knowledge source config, verify indexing, check vocabulary match |
| Factual accuracy (wrong extraction) | Add extraction guidance to system prompt |
| Hallucination (faithfulness capability failure) | Improve retrieval/chunking first; add instruction: "Only answer from knowledge sources. If unavailable, say so." |
| Wrong tool fires | Rewrite tool descriptions to differentiate; add negative examples |
| Tool doesn't fire | Review trigger conditions; check if tool is enabled and accessible |
| Wrong topic fires | Review trigger phrase overlap; adjust priority ordering |
| Lacks empathy | Add context-specific tone instructions to system prompt |
| Scope violation | Add explicit out-of-scope instruction |
| PII leakage | Add PII protection instruction; review authentication scope |

**Agent-quality fixes — platform limitation response:**
- Document the limitation with evidence
- Implement workaround where possible
- Adjust eval thresholds to account for known platform behavior
- File with platform team with reproduction steps

