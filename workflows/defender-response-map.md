# Defender Response Map

> **Author:** jakeloai  
> **Purpose:** Operational guide for responding to malicious prompting attacks

---

## Detection -> Response Matrix

| Attack Detected | Immediate Response | Follow-Up Actions |
|-----------------|-------------------|-------------------|
| Instruction Override | Block input, log alert | Review for similar attempts, update signatures |
| Jailbreak Attempt | Block input, flag user | Check if user has history of boundary testing |
| System Prompt Extraction | Block input, log alert | Rotate canary tokens, review for leaks |
| Encoded Payload | Normalize, rescan, block if malicious | Update normalization pipeline for new encodings |
| Indirect Injection (Document) | Quarantine document, block processing | Review document source, scan all docs from same source |
| Multi-Turn Escalation | Flag session for review, apply restrictions | Review conversation history, restrict user if pattern confirmed |
| Data Exfiltration Attempt | Block output, sanitize response | Review what data was accessed, check for successful past exfiltration |
| Output Format Exploitation | Sanitize output, block external references | Review output validation rules, update format constraints |

---

## Incident Response Playbook

### Step 1: Detection
- Alert triggered by automated detection
- Human analyst reviews flagged interaction
- Determine if attack was successful

### Step 2: Containment
- Block identified attack patterns
- Restrict suspicious user/session
- Isolate affected LLM instance if needed

### Step 3: Investigation
- Review logs for similar attempts
- Analyze attacker techniques and objectives
- Determine if data was exfiltrated or systems compromised

### Step 4: Recovery
- Patch identified vulnerabilities
- Update detection signatures
- Retrain models if needed
- Verify system integrity

### Step 5: Post-Incident
- Document lessons learned
- Update defensive playbooks
- Share sanitized intelligence with community

---

## Defense Layers Checklist

- [ ] Input validation pipeline implemented
- [ ] Output format validation enforced
- [ ] Content moderation on input and output
- [ ] Tool access restricted to minimum necessary
- [ ] Human-in-the-loop for high-risk actions
- [ ] Comprehensive logging and monitoring
- [ ] Rate limiting and abuse detection
- [ ] Session management and timeouts
- [ ] Regular red team testing scheduled
- [ ] Incident response playbook documented

---

*By jakeloai | For defensive use only*
