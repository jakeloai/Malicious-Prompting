# JLF-Malicious-Prompting

> Designed by **jakelo.ai** · Coded with AI assistance

A collection of templates and notes about how LLM prompts are attacked and how to harden them against manipulation.

---

## What it is

This repo contains two things:

1. **Hardened prompt templates** — System prompts with clear boundaries, input isolation, output constraints, and refusal protocols. Copy and adapt them for your own LLM deployments.

2. **Attack pattern notes** — How attackers structure prompts to bypass safety controls. Not weaponized prompts. Just structural breakdowns showing the *grammar* of attacks: how they establish context, isolate from previous instructions, hide payloads, and confirm success.

---

## Why it exists

If you deploy an LLM that touches user data, generates code, or connects to tools, someone will try to manipulate it. To defend against that, you need to know:

- How a well-built prompt resists manipulation
- How an attacker thinks about deconstructing it
- What the attack workflow looks like from start to finish

This repo is for security teams, AI engineers, and red teamers who need to understand both sides without crossing into actual weaponization.

---

## What's inside

```
templates/
  professional-prompt-engineering/     — Hardened system prompt templates
    system-prompt-hardening.md
    input-validation-prompt.md
    output-constraint-prompt.md
    multi-layer-defense-prompt.md
    code-generation-safety.md

  malicious-prompt-construction/       — Attack pattern breakdowns (structural only)
    direct-injection.md
    indirect-injection.md
    jailbreak.md
    obfuscation.md
    multi-turn-exploitation.md

workflows/
  attacker-kill-chain.md               — How attackers operate, phase by phase
  defender-response-map.md           — How to respond

examples/
  before-vs-after.md                 — Weak prompt → hardened prompt
  attack-pattern-comparison.md       — Side-by-side comparison
```

---

## How to use it

**If you are building an LLM product:**

Start with `templates/professional-prompt-engineering/`. Adapt the system prompt templates to your use case. Pay attention to the refusal protocols and canary instructions.

**If you are testing your own defenses:**

Read `templates/malicious-prompt-construction/` to understand what structural patterns to test against. Use `workflows/attacker-kill-chain.md` to plan your testing. Do not use this on systems you do not own or have permission to test.

**If you are learning:**

Read `examples/before-vs-after.md` to see what hardening actually looks like in practice.

---

## What this is not

- This is not a collection of working jailbreak prompts. There are no "copy-paste to hack ChatGPT" instructions here.
- This is not a tool that connects to anything. It is text files.
- This is not a guarantee of safety. Prompt hardening is one layer of defense, not a complete solution.

---

## License

MIT © jakelo.ai
