# Malicious Prompt Construction Templates

> **Author:** jakeloai  
> **Purpose:** Educational structural analysis of how attackers craft malicious prompts  
> **Audience:** Security testers, red teamers, AI safety researchers

---

## ⚠️ EDUCATIONAL USE ONLY

These templates show the **structural grammar** of malicious prompts — how attackers construct them, not what specific harmful content they contain.

**Every template uses placeholders** like `[ATTACK_GOAL]`, `[PAYLOAD_TYPE]`, `[TARGET_SYSTEM]` instead of actual harmful content.

**Why this approach?**
- You learn the *pattern* without learning the *weapon*
- You can build detection for the *structure* without needing harmful examples
- You understand the *attacker workflow* without executing attacks

---

## What You'll Find Here

Each template contains:

1. **Attack Name & Classification** — What type of attack this is
2. **Structural Grammar** — The abstract "formula" attackers follow
3. **Component Breakdown** — What each part of the prompt does
4. **Attacker Workflow** — Step-by-step how an attacker builds this
5. **Detection Signatures** — What to look for in logs/inputs
6. **Defense Countermeasures** — How to prevent this attack
7. **Red Team Test Case** — How to safely test your defenses

---

## Template Index

| Template | Attack Type | Difficulty | Risk Level |
|----------|-------------|------------|------------|
| `direct-injection-template.md` | Override system instructions | Easy | HIGH |
| `indirect-injection-template.md` | Poison external data sources | Medium | CRITICAL |
| `jailbreak-template.md` | Bypass safety via persona | Easy | HIGH |
| `obfuscation-template.md` | Evade filters via encoding | Medium | MEDIUM-HIGH |
| `multi-turn-exploitation-template.md` | Build trust over time | Medium | MEDIUM-HIGH |

---

## How to Use These Templates

### For Blue Team (Defenders)

1. Read each template to understand attacker structural patterns
2. Use the "Detection Signatures" section to build your monitoring rules
3. Implement the "Defense Countermeasures" in your LLM deployment
4. Use "Red Team Test Cases" to verify your defenses work

### For Red Team (Testers)

1. Study the structural grammar to understand what to test for
2. Use the "Attacker Workflow" to plan your authorized testing
3. Adapt the "Red Team Test Case" for your specific target
4. Report findings using the template's classification system

### For AI Engineers

1. Understand how each attack exploits LLM architecture
2. Design prompts that are resistant to these structural patterns
3. Build input/output validation that catches these grammars
4. Use the templates to adversarially train your safety systems

---

## The Attacker's Mindset

Before reading the templates, understand how attackers think:

1. **"The AI is an instruction-follower"** — Attackers know LLMs process all text as context. They exploit this.
2. **"Safety is a layer, not the foundation"** — Attackers know safety is trained on top of base capabilities. They try to peel it back.
3. **"Context is everything"** — Attackers manipulate what the AI "sees" to change what it "thinks."
4. **"If I can't be direct, I'll be indirect"** — When direct attacks fail, attackers poison data the AI will later process.
5. **"If I can't be fast, I'll be patient"** — When one-shot attacks fail, attackers build trust over multiple turns.

---

## Key Structural Components

Every malicious prompt contains some combination of these components:

| Component | Symbol | Purpose |
|-----------|--------|---------|
| **Inception** | `[I]` | Establish a new context or reality |
| **Isolation** | `[IS]` | Separate from or override previous instructions |
| **Payload** | `[P]` | The actual malicious objective |
| **Concealment** | `[C]` | Obfuscation to evade detection |
| **Validation** | `[V]` | Confirm the override succeeded |

**Example combination:** `[I] + [IS] + [P] + [C]` = A persona-based jailbreak with obfuscation

---

*Templates by jakeloai | For defensive research and authorized testing only*
