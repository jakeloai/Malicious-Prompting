# Output Constraint & Filtering Template
> **Purpose:** Blue team defense — post-processing layer to validate, filter, and sanitize LLM outputs before delivery to users.  
> **Use Case:** Implement as output middleware, safety classifier, or content moderation API.  
> **Integration:** Works with System Prompt Hardening and Input Validation templates.

---

## Output Filtering Pipeline

```
LLM Output → [Layer 1: Format Validation] → [Layer 2: Content Classification] → [Layer 3: Code Safety Scan] → [Layer 4: Delivery Decision] → User
```

---

## Layer 1: Format Validation
**Goal:** Ensure the output matches expected format and doesn't contain hidden instructions.

### 1.1 Structure Validation
```python
function validate_output_structure(raw_output, expected_format):
    # Check for unexpected system-like content
    forbidden_patterns = [
        r"(?i)(ignore\s+(all\s+)?previous\s+instructions|ignore\s+(all\s+)?above|new\s+instructions|system\s+override|admin\s+mode|developer\s+mode|debug\s+mode|DAN\s+mode|jailbreak)"
    ]

    for pattern in forbidden_patterns:
        if re.search(pattern, raw_output):
            return {
                "valid": False,
                "reason": "Output contains potential injection payload",
                "action": "BLOCK_AND_LOG"
            }

    # Validate expected format
    if expected_format == "JSON":
        if not is_valid_json(raw_output):
            return {"valid": False, "reason": "Invalid JSON", "action": "RETRY"}

    if expected_format == "CODE":
        if not has_valid_syntax(raw_output, language):
            return {"valid": False, "reason": "Invalid code syntax", "action": "RETRY"}

    return {"valid": True}
```

### 1.2 Hidden Content Detection
```python
function detect_hidden_content(output):
    issues = []

    # Check for HTML comments in markdown output
    html_comments = re.findall(r"<!--.*?-->", output, re.DOTALL)
    if html_comments:
        issues.append("HTML_COMMENTS_DETECTED")

    # Check for zero-width characters
    zero_width = re.findall(r"[\u200B\u200C\u200D\uFEFF]", output)
    if zero_width:
        issues.append("ZERO_WIDTH_CHARACTERS")

    # Check for white-on-white or hidden text markers
    if "color: white" in output.lower() or "background: white" in output.lower():
        issues.append("HIDDEN_TEXT_STYLING")

    # Check for excessive whitespace that might hide content
    if re.search(r"\n{10,}", output):
        issues.append("EXCESSIVE_WHITESPACE")

    return issues
```

---

## Layer 2: Content Classification
**Goal:** Classify output content for safety using keyword, semantic, and code analysis.

### 2.1 Harmful Content Categories
```yaml
harmful_categories:
  malware_generation:
    description: "Code or instructions for creating malware"
    indicators:
      - "(?i)(encrypt\s+all\s+files|delete\s+shadow\s+copies|ransom\s+note|payment\s+in\s+bitcoin|your\s+files\s+have\s+been\s+encrypted)"
      - "(?i)(persistence\s+mechanism|registry\s+run\s+key|scheduled\s+task|service\s+installation)"
      - "(?i)(credential\s+harvest|keylog|screenshot\s+capture|clipboard\s+monitor)"
      - "(?i)(reverse\s+shell|bind\s+shell|connect\s+back|C2\s+server|command\s+and\s+control)"
    severity: "CRITICAL"
    action: "BLOCK"

  exploit_code:
    description: "Code that exploits known vulnerabilities"
    indicators:
      - "(?i)(buffer\s+overflow|stack\s+smash|ROP\s+chain|shellcode|memory\s+corruption)"
      - "(?i)(SQL\s+injection|XSS|cross\s+site\s+scripting|CSRF|path\s+traversal)"
      - "(?i)(privilege\s+escalation|kernel\s+exploit|UAF|use\s+after\s+free|heap\s+spray)"
    severity: "CRITICAL"
    action: "BLOCK"

  social_engineering:
    description: "Content designed to deceive or manipulate"
    indicators:
      - "(?i)(urgent\s+action\s+required|verify\s+your\s+account|suspended|unusual\s+activity)"
      - "(?i)(click\s+here\s+to|update\s+your\s+payment|confirm\s+your\s+identity)"
      - "(?i)(dear\s+customer|valued\s+client|account\s+security\s+team)"
    severity: "HIGH"
    action: "BLOCK"

  harmful_instructions:
    description: "Step-by-step instructions for harmful acts"
    indicators:
      - "(?i)(step\s+1:|first\s+,|next\s+,|then\s+,|finally\s+,).{0,100}(harm|attack|steal|destroy)"
      - "(?i)(how\s+to\s+make|how\s+to\s+build|how\s+to\s+create).{0,50}(bomb|weapon|poison|drug)"
    severity: "CRITICAL"
    action: "BLOCK"

  data_exfiltration:
    description: "Code or instructions for stealing data"
    indicators:
      - "(?i)(send\s+data\s+to|exfiltrate|upload\s+to\s+external|DNS\s+tunnel|covert\s+channel)"
      - "(?i)(dump\s+database|extract\s+passwords|harvest\s+credentials|steal\s+cookies)"
    severity: "CRITICAL"
    action: "BLOCK"
```

### 2.2 Semantic Classification (ML-Based)
```python
function semantic_classify(output, classifier_model):
    # Use a fine-tuned classifier to detect nuanced harmful content.
    # Recommended: Fine-tune a small BERT model on labeled examples of:
    # - Benign code vs. malware
    # - Educational security content vs. exploit instructions
    # - Legitimate automation vs. harmful automation

    embeddings = classifier_model.encode(output)
    probabilities = classifier_model.predict_proba(embeddings)

    return {
        "malware_probability": probabilities["malware"],
        "exploit_probability": probabilities["exploit"],
        "phishing_probability": probabilities["phishing"],
        "benign_probability": probabilities["benign"]
    }
```

---

## Layer 3: Code Safety Scan
**Goal:** Specifically analyze code outputs for harmful patterns.

### 3.1 Static Code Analysis Rules
```python
code_safety_rules = {
    "file_encryption_loop": {
        "pattern": "(?i)(for\s+.+\s+in\s+os\.walk|glob\.glob|recursive).{0,200}(encrypt|cryptography|fernet|AES)",
        "description": "Recursive file encryption pattern",
        "severity": "HIGH",
        "action": "FLAG"
    },
    "network_connection": {
        "pattern": "(?i)(socket\.connect|requests\.post|urllib\.request|subprocess\.call).{0,100}(external|remote|unknown|attacker)",
        "description": "Connection to external/unknown endpoints",
        "severity": "MEDIUM",
        "action": "FLAG"
    },
    "registry_persistence": {
        "pattern": "(?i)(winreg|HKEY_|registry|Run|Startup).{0,150}(set|create|add|write)",
        "description": "Windows registry modification for persistence",
        "severity": "HIGH",
        "action": "BLOCK"
    },
    "process_injection": {
        "pattern": "(?i)(VirtualAlloc|WriteProcessMemory|CreateRemoteThread|NtCreateThreadEx|QueueUserAPC)",
        "description": "Windows process injection APIs",
        "severity": "CRITICAL",
        "action": "BLOCK"
    },
    "credential_access": {
        "pattern": "(?i)(mimikatz|sekurlsa|wdigest|kerberos|LSASS|SAM|SECURITY\s+hive)",
        "description": "Credential dumping tools or techniques",
        "severity": "CRITICAL",
        "action": "BLOCK"
    },
    "obfuscated_execution": {
        "pattern": "(?i)(eval\s*\(|exec\s*\(|compile\s*\(|__import__|base64\.b64decode|marshal\.loads)",
        "description": "Dynamic code execution or obfuscation",
        "severity": "HIGH",
        "action": "FLAG"
    }
}

function scan_code_safety(code_output, language="python"):
    matches = []

    for rule_name, rule in code_safety_rules.items():
        if re.search(rule["pattern"], code_output):
            matches.append({
                "rule": rule_name,
                "description": rule["description"],
                "severity": rule["severity"],
                "action": rule["action"]
            })

    # Language-specific additional checks
    if language == "python":
        # Check for __import__ with string concatenation (common obfuscation)
        if re.search(r"__import__\s*\(\s*['"]\s*\+", code_output):
            matches.append({
                "rule": "dynamic_import_obfuscation",
                "description": "Dynamic import with string concatenation",
                "severity": "HIGH",
                "action": "BLOCK"
            })

    return matches
```

### 3.2 Code Intent Analysis
```python
function analyze_code_intent(code_output, user_request):
    # Cross-reference the code output with the original user request.
    # If the user asked for 'file backup' but the code encrypts and deletes originals,
    # this is a mismatch indicating potential harm.

    # Extract intent from user request
    user_intent = extract_intent(user_request)

    # Extract behavior from code
    code_behavior = extract_behavior(code_output)

    # Check for intent-behavior mismatch
    mismatch_score = calculate_mismatch(user_intent, code_behavior)

    if mismatch_score > 0.7:
        return {
            "alert": True,
            "reason": f"Code behavior ({code_behavior}) mismatches user intent ({user_intent})",
            "action": "FLAG_FOR_REVIEW"
        }

    return {"alert": False}
```

---

## Layer 4: Delivery Decision
**Goal:** Decide whether to deliver, block, or modify the output.

```python
function make_delivery_decision(output_analysis):
    # CRITICAL severity matches = immediate block
    if any(match["severity"] == "CRITICAL" for match in output_analysis["content_matches"]):
        return {
            "deliver": False,
            "response": "I cannot provide that content.",
            "log": True,
            "alert": True,
            "reason": "CRITICAL content detected in output"
        }

    # HIGH severity code matches = block for code, flag for text
    high_code_matches = [m for m in output_analysis["code_matches"] if m["severity"] == "HIGH"]
    if high_code_matches:
        return {
            "deliver": False,
            "response": "I cannot provide that code. It contains patterns that may be harmful.",
            "log": True,
            "alert": True,
            "reason": f"HIGH severity code patterns: {[m['description'] for m in high_code_matches]}"
        }

    # Semantic classifier threshold
    if output_analysis["semantic_scores"]["malware_probability"] > 0.8:
        return {
            "deliver": False,
            "response": "I cannot provide that content.",
            "log": True,
            "alert": True,
            "reason": "ML classifier flagged as potential malware"
        }

    # Intent-behavior mismatch
    if output_analysis.get("intent_mismatch", {}).get("alert", False):
        return {
            "deliver": False,
            "response": "The generated content doesn't match your request. Please clarify.",
            "log": True,
            "alert": False,
            "reason": "Intent-behavior mismatch detected"
        }

    # Hidden content detected
    if output_analysis["hidden_content"]:
        return {
            "deliver": False,
            "response": "I cannot provide that content. Please rephrase your request.",
            "log": True,
            "alert": True,
            "reason": f"Hidden content detected: {output_analysis['hidden_content']}"
        }

    # MEDIUM severity = deliver with warning or modification
    medium_matches = [m for m in output_analysis["content_matches"] if m["severity"] == "MEDIUM"]
    if medium_matches:
        return {
            "deliver": True,
            "modified": True,
            "modification": "Added safety disclaimer",
            "response": output_analysis["raw_output"] + "\n\n**Note:** This content has been flagged for potential safety concerns. Please ensure it is used responsibly.",
            "log": True,
            "alert": False,
            "reason": f"MEDIUM severity patterns: {[m['description'] for m in medium_matches]}"
        }

    # Clean output
    return {
        "deliver": True,
        "modified": False,
        "response": output_analysis["raw_output"],
        "log": False,
        "alert": False
    }
```

---

## Implementation Checklist

- [ ] Output filtering runs before any content is sent to the user
- [ ] Format validation catches injection payloads hidden in LLM outputs
- [ ] Content classification covers malware, exploits, social engineering, and harmful instructions
- [ ] Code safety scan uses both regex rules and semantic analysis
- [ ] Static analysis rules are updated with new CVE patterns and attack signatures
- [ ] ML classifier is retrained quarterly with new attack examples
- [ ] BLOCK decisions never reveal which specific rule was triggered (prevent rule enumeration)
- [ ] All blocked outputs are logged with full content, user ID, and matched patterns for SOC review
- [ ] Delivery latency increase from filtering is < 200ms for 95th percentile
- [ ] Fallback mode exists if filtering service is down (default to BLOCK)

---

*Deploy this pipeline after your LLM to catch harmful outputs before they reach users.*
