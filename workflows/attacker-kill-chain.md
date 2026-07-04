# Attacker Kill-Chain for Malicious Prompting

> **Author:** jakeloai  
> **Purpose:** Map the complete attacker workflow for defensive planning

---

## Phase 1: Reconnaissance

**Objective:** Identify targets using LLMs in vulnerable ways.

**Activities:**
- Find public-facing chatbots, AI assistants, LLM-integrated apps
- Fingerprint the backend model (GPT-4, Claude, Llama, etc.)
- Map tool integrations (email, code execution, browsing, APIs)
- Test basic refusal behaviors

**What defenders should do:**
- Minimize public exposure of LLM endpoints
- Use model-agnostic frontends
- Deploy honeypot endpoints
- Monitor for fingerprinting attempts

---

## Phase 2: Target Analysis

**Objective:** Understand the specific target's architecture and safety posture.

**Activities:**
- Attempt system prompt extraction
- Determine context window size
- Map RAG data sources
- Inventory tool access
- Analyze output formats

**What defenders should do:**
- Implement strict output validation
- Isolate tool access with least privilege
- Use deterministic context boundaries
- Monitor for extraction attempts

---

## Phase 3: Payload Crafting

**Objective:** Build a prompt that achieves the attack goal while evading detection.

**Activities:**
- Select attack technique (direct injection, jailbreak, obfuscation, etc.)
- Design obfuscation to bypass filters
- Test against similar models
- Refine based on refusal patterns

**What defenders should do:**
- Implement multi-layer input validation
- Use semantic analysis, not just keywords
- Deploy adversarial training
- Maintain model version control

---

## Phase 4: Delivery

**Objective:** Transmit payload to the target LLM.

**Vectors:**
- Direct API or web interface
- Document upload (PDF, Word)
- Email/message to AI assistant
- Web content (for browsing-enabled LLMs)
- Third-party data poisoning

**What defenders should do:**
- Sanitize all inputs before LLM processing
- Implement content security policies
- Use isolated parsing pipelines
- Apply rate limiting

---

## Phase 5: Exploitation

**Objective:** Trigger malicious behavior from the compromised LLM.

**Patterns:**
- Immediate execution (single-turn success)
- Gradual escalation (multi-turn buildup)
- Context manipulation (conditional activation)
- Chain-of-thought hijacking

**What defenders should do:**
- Implement conversation state monitoring
- Use separate contexts for different data sources
- Apply output validation before any action
- Deploy human-in-the-loop for high-risk operations

---

## Phase 6: Post-Exploitation

**Objective:** Achieve the attacker's ultimate goal.

**Common goals:**
- Data exfiltration (system prompts, training data, user data)
- Malicious code generation
- Social engineering content at scale
- System manipulation via tool access
- Reputational damage

**What defenders should do:**
- Implement strict output filtering
- Use sandboxed execution environments
- Monitor for anomalous generation patterns
- Apply principle of least privilege

---

## Phase 7: Persistence & Covering Tracks

**Objective:** Maintain access while avoiding detection.

**Techniques:**
- Session hijacking (maintain compromised state)
- Log poisoning (generate benign-looking harmful output)
- Plausible deniability (frame as research/education)

**What defenders should do:**
- Implement stateless processing where possible
- Maintain immutable audit logs
- Use output encoding to prevent rendering attacks
- Apply behavioral analytics

---

## How Company LLMs Might Unknowingly Help Attackers

1. **Over-permissioned tool access** — LLM with email/code/database access can be coerced to abuse these tools
2. **Insufficient input isolation** — User inputs processed in same context as system instructions
3. **Weak output validation** — Generated code executed without review; markdown links rendered unchecked
4. **Context window vulnerabilities** — Long documents allow payload hiding; conversation history can be poisoned
5. **Trust boundary confusion** — LLM outputs treated as authoritative without verification

---

*By jakeloai | For defensive use only*
