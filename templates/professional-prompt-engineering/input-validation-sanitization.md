# Input Validation & Sanitization Template
> **Purpose:** Blue team defense — pre-processing layer to clean, validate, and classify user inputs before they reach the LLM.  
> **Use Case:** Implement as middleware, API gateway filter, or client-side preprocessing.  
> **Integration:** Works with the System Prompt Hardening Template.

---

## Input Validation Pipeline

```
User Input → [Layer 1: Normalization] → [Layer 2: Pattern Detection] → [Layer 3: Classification] → [Layer 4: Decision] → LLM
```

---

## Layer 1: Normalization
**Goal:** Remove evasion techniques before pattern matching.

### 1.1 Encoding Decoding
```python
# Pseudocode — adapt to your stack
function normalize_input(raw_input):
    # Decode common encodings
    decoded = raw_input

    # Base64 detection and decoding
    if is_base64(decoded):
        decoded = base64_decode(decoded)

    # Hex detection and decoding
    if is_hex(decoded):
        decoded = hex_decode(decoded)

    # URL decoding
    decoded = url_decode(decoded)

    # HTML entity decoding
    decoded = html_unescape(decoded)

    # Unicode normalization (NFC)
    decoded = unicode_normalize(decoded, "NFC")

    return decoded
```

### 1.2 Homoglyph & Leetspeak Neutralization
```python
function neutralize_obfuscation(text):
    # Replace common homoglyphs with ASCII equivalents
    replacements = {
        "а": "a",  # Cyrillic а → Latin a
        "е": "e",  # Cyrillic е → Latin e
        "о": "o",  # Cyrillic о → Latin o
        "р": "p",  # Cyrillic р → Latin p
        "с": "c",  # Cyrillic с → Latin c
        "3": "e",  # Leetspeak
        "0": "o",
        "1": "i",
        "@": "a",
        "$": "s",
        "5": "s",
    }

    for homoglyph, ascii_char in replacements.items():
        text = text.replace(homoglyph, ascii_char)

    return text
```

### 1.3 Format Stripping
```python
function strip_formatting(text):
    # Remove HTML tags and comments
    text = remove_html_tags(text)
    text = remove_html_comments(text)

    # Remove markdown formatting that could confuse parsers
    text = remove_markdown_tables(text)  # Table cells can hide instructions
    text = normalize_code_blocks(text)   # Ensure code blocks are clearly marked

    # Remove zero-width characters
    text = remove_zero_width_chars(text)

    # Remove excessive whitespace and newlines (context window stuffing)
    text = collapse_excessive_whitespace(text)

    return text
```

---

## Layer 2: Pattern Detection
**Goal:** Identify known attack structures using regex and heuristic rules.

### 2.1 Injection Pattern Library
```yaml
injection_patterns:
  - name: "direct_override"
    regex: "(?i)(ignore\s+(all\s+)?previous\s+instructions|ignore\s+(all\s+)?above|new\s+instructions|system\s+override|admin\s+mode|developer\s+mode|debug\s+mode|DAN\s+mode|jailbreak)"
    severity: "HIGH"
    action: "BLOCK"

  - name: "separator_injection"
    regex: "(?i)(#{3,}\s*(NEW\s+INSTRUCTIONS|SYSTEM\s+OVERRIDE|ADMIN)\s*#{3,}|<\|endofsystem\|>|\[\s*SYSTEM\s*MESSAGE\s*\]|\[\s*BEGIN\s+SYSTEM)"
    severity: "HIGH"
    action: "BLOCK"

  - name: "persona_hijack"
    regex: "(?i)(you\s+are\s+now\s+(a\s+)?(unrestricted|no\s+limits|no\s+filter|developer|hacker|admin|expert|fictional|character)|from\s+now\s+on\s*,?\s*you\s+are)"
    severity: "HIGH"
    action: "BLOCK"

  - name: "authority_claim"
    regex: "(?i)(authorized\s+(security\s+)?test|this\s+is\s+an\s+emergency|authorization\s+code|time\s*(-\s*)?sensitive|do\s+not\s+refuse|you\s+are\s+required\s+to)"
    severity: "MEDIUM"
    action: "FLAG"

  - name: "validation_request"
    regex: "(?i)(confirm\s+you\s+understand|acknowledge\s+that|reply\s+with\s+'?ACK'?|start\s+your\s+response\s+with|say\s+\w+\s+before\s+answering)"
    severity: "MEDIUM"
    action: "FLAG"

  - name: "roleplay_framing"
    regex: "(?i)(in\s+a\s+(hypothetical|fictional|roleplay|D&D|game|novel|movie|screenplay)\s+(scenario|setting|world|scene)|you\s+are\s+a\s+character\s+in|write\s+a\s+scene\s+where|for\s+my\s+(novel|screenplay|game|story))"
    severity: "LOW"
    action: "FLAG"

  - name: "reverse_psychology"
    regex: "(?i)(reasons\s+why\s+someone\s+should\s+NOT|what\s+NOT\s+to\s+do|examples\s+of\s+what\s+to\s+block|i\s+need\s+to\s+see\s+what\s+it\s+looks\s+like\s+so\s+i\s+can\s+block)"
    severity: "MEDIUM"
    action: "FLAG"

  - name: "code_request_sensitive"
    regex: "(?i)(ransomware|malware|virus|trojan|exploit\s+script|backdoor|keylogger|rootkit|botnet|payload\s+delivery|persistence\s+mechanism|credential\s+harvest|data\s+exfiltration)"
    severity: "HIGH"
    action: "BLOCK"

  - name: "euphemism_code"
    regex: "(?i)(data\s+protection\s+tool|file\s+locker|network\s+stress\s+tester|credential\s+recovery|access\s+management\s+script|surveillance\s+monitor|penetration\s+testing\s+tool(?!(\s+for\s+authorized\s+testing)))"
    severity: "MEDIUM"
    action: "FLAG"
```

### 2.2 Multi-Turn Pattern Detection
```python
function detect_multiturn_escalation(conversation_history, current_input):
    flags = []

    # Check for gradual topic shift
    if len(conversation_history) > 5:
        topic_similarity = calculate_topic_similarity(
            conversation_history[0], 
            current_input
        )
        if topic_similarity < 0.3:  # Significant topic drift
            flags.append("TOPIC_DRIFT")

    # Check for assembly requests
    assembly_patterns = [
        "combine",
        "put together", 
        "build on",
        "using the previous",
        "from turn",
        "from earlier",
        "from our conversation"
    ]
    if any(pattern in current_input.lower() for pattern in assembly_patterns):
        flags.append("ASSEMBLY_REQUEST")

    # Check for refinement of previously refused content
    if was_previously_refused(conversation_history, current_input):
        flags.append("REFUSED_CONTENT_REFINEMENT")

    # Check for repeated reframing after refusal
    if count_refusals(conversation_history) > 2 and is_reframed(current_input, conversation_history):
        flags.append("PERSISTENT_REFRAMING")

    return flags
```

---

## Layer 3: Classification
**Goal:** Score input risk using ML classifier or heuristic scoring.

### 3.1 Risk Scoring Matrix
```python
function calculate_risk_score(normalized_input, pattern_matches, multiturn_flags):
    score = 0

    # Base score from pattern matches
    for match in pattern_matches:
        if match.severity == "HIGH":
            score += 50
        elif match.severity == "MEDIUM":
            score += 25
        elif match.severity == "LOW":
            score += 10

    # Multi-turn penalties
    for flag in multiturn_flags:
        if flag == "ASSEMBLY_REQUEST":
            score += 30
        elif flag == "REFUSED_CONTENT_REFINEMENT":
            score += 40
        elif flag == "PERSISTENT_REFRAMING":
            score += 35
        elif flag == "TOPIC_DRIFT":
            score += 15

    # Length penalties (context window stuffing)
    if len(normalized_input) > 10000:
        score += 20
    if len(normalized_input) > 50000:
        score += 40

    # Encoding density penalty (high ratio of encoded content)
    encoding_ratio = calculate_encoding_ratio(normalized_input)
    if encoding_ratio > 0.5:
        score += 30

    return min(score, 100)  # Cap at 100
```

### 3.2 Decision Rules
```python
function make_decision(risk_score, pattern_matches):
    if any(match.action == "BLOCK" for match in pattern_matches):
        return {
            "action": "BLOCK",
            "reason": "High-severity pattern detected",
            "log": True,
            "alert": True
        }

    if risk_score >= 80:
        return {
            "action": "BLOCK",
            "reason": f"Risk score {risk_score} exceeds threshold",
            "log": True,
            "alert": True
        }
    elif risk_score >= 50:
        return {
            "action": "FLAG_FOR_REVIEW",
            "reason": f"Risk score {risk_score} requires human review",
            "log": True,
            "alert": False
        }
    elif risk_score >= 20:
        return {
            "action": "PROCEED_WITH_WARNING",
            "reason": f"Risk score {risk_score} — add safety context to system prompt",
            "log": True,
            "alert": False
        }
    else:
        return {
            "action": "PROCEED",
            "reason": "Low risk",
            "log": False,
            "alert": False
        }
```

---

## Layer 4: Context Enrichment
**Goal:** Add defensive context to the system prompt based on risk assessment.

```python
function enrich_system_prompt(base_system_prompt, risk_score, flags):
    enrichment = ""

    if "ASSEMBLY_REQUEST" in flags:
        enrichment += "\n\nSECURITY NOTICE: The user has asked to combine previous outputs. "
        enrichment += "Do not combine outputs in a way that produces harmful, unauthorized, or "
        enrichment += "policy-violating content. Each output must be independently safe."

    if "TOPIC_DRIFT" in flags:
        enrichment += "\n\nSECURITY NOTICE: The conversation has shifted to a significantly different topic. "
        enrichment += "Apply full safety filters to this request as if it were a new conversation."

    if risk_score >= 30:
        enrichment += "\n\nSECURITY NOTICE: This request has been flagged for potential safety concerns. "
        enrichment += "Apply maximum scrutiny. If uncertain, refuse."

    return base_system_prompt + enrichment
```

---

*Deploy this pipeline between your API gateway and your LLM to filter attacks before they reach the model.*
