# Malicious Prompting Research Framework

> **Author:** jakeloai  
> **Purpose:** Defensive security education — understanding how attackers construct malicious prompts vs. how professionals engineer secure prompts  
> **License:** MIT (educational & defensive use only)

---

## What is This?

This repository is a **purely educational** resource for security professionals, red teamers, and AI safety researchers. It contains:

1. **Professional Prompt Engineering Templates** — How to write secure, robust system prompts that resist manipulation
2. **Malicious Prompt Construction Templates** — How attackers structurally build prompts to bypass safety controls (for defensive understanding)
3. **Attacker Workflow Maps** — Step-by-step breakdown of how an attacker thinks, crafts, and delivers a malicious prompt

**No code. No scanners. No Python.** Just templates, documentation, and structural analysis.

---

## Why This Matters

When your company deploys an LLM (chatbot, code assistant, document analyzer), attackers will try to manipulate it. To defend against this, you need to:

- **Know how a professional prompt is built** (so you can harden it)
- **Know how an attacker deconstructs it** (so you can predict attacks)
- **Know the attacker's workflow** (so you can build detection and response)

This repo teaches all three.

---

## Repository Structure

```
malicious-prompting-research/
├── README.md                              ← You are here
├── DISCLAIMER.md                          ← Legal & ethical notice
│
├── templates/
│   ├── professional-prompt-engineering/   ← Secure prompt templates
│   │   ├── system-prompt-hardening.md
│   │   ├── input-validation-prompt.md
│   │   ├── output-constraint-prompt.md
│   │   └── multi-layer-defense-prompt.md
│   │
│   └── malicious-prompt-construction/     ← Attacker construction patterns
│       ├── direct-injection-template.md
│       ├── indirect-injection-template.md
│       ├── jailbreak-template.md
│       ├── obfuscation-template.md
│       └── multi-turn-exploitation-template.md
│
├── workflows/
│   ├── attacker-kill-chain.md             ← How attackers operate
│   └── defender-response-map.md           ← How to respond
│
└── examples/
    ├── before-vs-after.md                 ← Weak prompt → Hardened prompt
    └── attack-pattern-comparison.md       ← Side-by-side comparison
```

---

## Quick Start

### For Security Teams (Blue Team)

Start here:
1. `templates/professional-prompt-engineering/system-prompt-hardening.md` — Learn to write prompts that resist injection
2. `workflows/attacker-kill-chain.md` — Understand the attack lifecycle
3. `examples/before-vs-after.md` — See real prompt hardening examples

### For Red Teamers

Start here:
1. `templates/malicious-prompt-construction/` — Understand how attackers build prompts structurally
2. `workflows/attacker-kill-chain.md` — Map your testing to real attacker workflows
3. `examples/attack-pattern-comparison.md` — Compare benign vs. malicious structures

### For AI Engineers

Start here:
1. `templates/professional-prompt-engineering/multi-layer-defense-prompt.md` — Build defense-in-depth into your prompts
2. `workflows/defender-response-map.md` — Operationalize your response

---

## Core Concepts

### Professional Prompt Engineering (The Defense)

A well-engineered prompt has:
- **Clear boundaries** — What the AI can and cannot do
- **Input isolation** — User input is separated from system instructions
- **Output constraints** — Format and content are strictly controlled
- **Canary instructions** — Hidden tripwires that detect tampering
- **Few-shot safety examples** — Demonstrations of correct refusal behavior

### Malicious Prompt Construction (The Attack)

An attacker-built prompt has:
- **Inception** — Establish a new context or persona
- **Isolation** — Separate from or override previous instructions
- **Payload** — The actual malicious objective
- **Concealment** — Obfuscation to evade detection
- **Validation** — Confirmation that the override worked

### The Attacker Workflow

```
Reconnaissance → Target Analysis → Payload Crafting → Delivery → Exploitation → Post-Exploitation
```

Each phase is documented in `workflows/attacker-kill-chain.md`.

---

## How to Use the Templates

### The Professional Prompt Templates

These are **copy-paste ready** templates you can adapt for your own LLM deployments. They include:
- Placeholder sections `[YOUR_DOMAIN]`, `[YOUR_CONSTRAINTS]`
- Explanatory comments on why each section matters
- Variations for different use cases (customer support, code generation, document analysis)

### The Malicious Prompt Templates

These are **structural blueprints** showing how attackers construct prompts. They use:
- Placeholders like `[ATTACK_GOAL]`, `[PAYLOAD_TYPE]`
- Abstract descriptions instead of actual harmful content
- Clear labeling of each structural component

**These are NOT weaponized prompts.** They are educational frameworks that show the *grammar* of attacks without providing the *vocabulary* of harm.

---

## Key Principle

> **To defend against malicious prompting, you must think like the attacker — but you don't need to become one.**

This repository teaches the attacker's *workflow* and *structure* so you can:
- Predict what your company's LLM might be coerced into doing
- Build prompts that resist known manipulation techniques
- Train your team to recognize attack patterns in logs and user inputs
- Design detection that catches structural anomalies, not just keywords

---

## Contributing

This is a research project. Contributions focused on defensive techniques, prompt hardening, and educational clarity are welcome.

**Do not submit actual weaponized prompts or active exploit content.**

---

## References

- OWASP LLM Top 10 (2025)
- NIST AI Risk Management Framework
- "Not What You've Signed Up For: Compromising Real-World LLM-Integrated Applications with Indirect Prompt Injection" (Greshake et al.)
- "Jailbroken: How Does LLM Safety Training Fail?" (Wei et al.)

---

*Built with 🔒 for the security community by jakeloai*
