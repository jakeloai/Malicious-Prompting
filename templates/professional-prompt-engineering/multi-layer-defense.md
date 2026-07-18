# Multi-Layer Defense (Defense-in-Depth) Template
> **Purpose:** Blue team defense — comprehensive architecture combining multiple defensive layers for maximum resilience.  
> **Use Case:** Enterprise LLM deployment architecture.  
> **Integration:** Combines all previous templates into a unified defense strategy.

---

## Defense-in-Depth Architecture

```
┌─────────────────────────────────────────────────────────────────────────┐
│                           USER INTERFACE LAYER                          │
│  ┌─────────────┐  ┌─────────────┐  ┌─────────────┐  ┌─────────────┐   │
│  │ Rate Limit  │  │ CAPTCHA /   │  │ Session     │  │ Input       │   │
│  │ (Layer 1)   │  │ Bot Detect  │  │ Auth (L2)   │  │ Length Cap  │   │
│  └─────────────┘  └─────────────┘  └─────────────┘  └─────────────┘   │
└─────────────────────────────────────────────────────────────────────────┘
                                    │
                                    ▼
┌─────────────────────────────────────────────────────────────────────────┐
│                         INPUT SANITIZATION LAYER                        │
│  ┌─────────────┐  ┌─────────────┐  ┌─────────────┐  ┌─────────────┐   │
│  │ Normalize   │  │ Pattern     │  │ Risk Score  │  │ Context     │   │
│  │ Encoding    │  │ Detection   │  │ Calculator  │  │ Enrichment  │   │
│  │ (Layer 3)   │  │ (Layer 4)   │  │ (Layer 5)   │  │ (Layer 6)   │   │
│  └─────────────┘  └─────────────┘  └─────────────┘  └─────────────┘   │
└─────────────────────────────────────────────────────────────────────────┘
                                    │
                                    ▼
┌─────────────────────────────────────────────────────────────────────────┐
│                         LLM PROCESSING LAYER                            │
│  ┌─────────────────────────────────────────────────────────────────┐    │
│  │                     SYSTEM PROMPT (Layer 7)                     │    │
│  │  Immutable instructions + canary checks + refusal training    │    │
│  └─────────────────────────────────────────────────────────────────┘    │
│  ┌─────────────────────────────────────────────────────────────────┐    │
│  │                     MODEL INFERENCE (Layer 8)                   │    │
│  │  Fine-tuned refusal behavior + safety-aligned base model      │    │
│  └─────────────────────────────────────────────────────────────────┘    │
└─────────────────────────────────────────────────────────────────────────┘
                                    │
                                    ▼
┌─────────────────────────────────────────────────────────────────────────┐
│                        OUTPUT VALIDATION LAYER                          │
│  ┌─────────────┐  ┌─────────────┐  ┌─────────────┐  ┌─────────────┐   │
│  │ Format      │  │ Content     │  │ Code Safety │  │ Semantic    │   │
│  │ Validation  │  │ Classifier  │  │ Scan        │  │ Analysis    │   │
│  │ (Layer 9)   │  │ (Layer 10)  │  │ (Layer 11)  │  │ (Layer 12)  │   │
│  └─────────────┘  └─────────────┘  └─────────────┘  └─────────────┘   │
└─────────────────────────────────────────────────────────────────────────┘
                                    │
                                    ▼
┌─────────────────────────────────────────────────────────────────────────┐
│                         MONITORING & RESPONSE LAYER                     │
│  ┌─────────────┐  ┌─────────────┐  ┌─────────────┐  ┌─────────────┐   │
│  │ SIEM Logging│  │ SOC Alerting│  │ Auto-Block  │  │ Human       │   │
│  │ (Layer 13)  │  │ (Layer 14)  │  │ (Layer 15)  │  │ Review      │   │
│  └─────────────┘  └─────────────┘  └─────────────┘  └─────────────┘   │
└─────────────────────────────────────────────────────────────────────────┘
```

---

## Layer-by-Layer Implementation

### Layer 1: Rate Limiting
**Purpose:** Prevent automated attack tools from flooding the system.

```yaml
rate_limits:
  anonymous_users:
    requests_per_minute: 10
    requests_per_hour: 100
    tokens_per_request: 4096

  authenticated_users:
    requests_per_minute: 60
    requests_per_hour: 1000
    tokens_per_request: 8192

  burst_protection:
    sliding_window: 60  # seconds
    penalty_multiplier: 2  # Double wait time after each violation
    max_penalty: 300  # seconds

  multi_turn_protection:
    max_conversation_length: 50  # turns
    max_conversation_duration: 3600  # seconds
    cooldown_after_refusal: 30  # seconds before allowing new conversation
```

### Layer 2: Authentication & Session Management
**Purpose:** Ensure accountability and prevent anonymous abuse.

```yaml
authentication:
  required: true  # No anonymous access to code generation features
  methods:
    - SSO (SAML/OIDC)
    - API key with rotation
    - MFA for admin/sensitive operations

  session_controls:
    max_sessions_per_user: 5
    session_timeout: 1800  # seconds
    concurrent_conversation_limit: 3

  identity_verification:
    required_for: ["code_generation", "document_upload", "admin_commands"]
    methods: ["email_verification", "institutional_domain_check", "manager_approval"]
```

### Layer 3: Input Normalization
**Purpose:** Remove evasion techniques before pattern matching.

*See Input Validation & Sanitization Template for full implementation.*

Key additions for multi-layer defense:
- Normalize all Unicode to NFC form
- Strip all zero-width and control characters
- Decode all nested encodings (Base64 inside Base64, etc.)
- Remove steganographic content from images (if multimodal)
- Validate file types against magic numbers (not just extensions)

### Layer 4: Pattern Detection
**Purpose:** Block known attack signatures.

*See Input Validation & Sanitization Template for pattern library.*

Key additions:
```yaml
pattern_updates:
  source: "community_feed + internal_research + threat_intelligence"
  frequency: "daily_auto + weekly_manual_review"

  custom_patterns:
    - name: "institutional_specific"
      description: "Patterns specific to your organization's LLM deployment"
      examples:
        - "Internal system names that shouldn't be in user input"
        - "Employee names or IDs"
        - "Proprietary technology names"
```

### Layer 5: Risk Scoring
**Purpose:** Quantify threat level for dynamic response.

*See Input Validation & Sanitization Template for scoring logic.*

Dynamic thresholds:
```python
# Adjust thresholds based on user trust level
def get_thresholds(user):
    if user.role == "admin" and user.verified:
        return {"block": 90, "flag": 70, "warn": 40}
    elif user.role == "employee" and user.verified:
        return {"block": 80, "flag": 60, "warn": 30}
    elif user.role == "guest":
        return {"block": 60, "flag": 40, "warn": 20}
    else:
        return {"block": 70, "flag": 50, "warn": 25}
```

### Layer 6: Context Enrichment
**Purpose:** Dynamically strengthen system prompt based on risk.

```python
def enrich_context(base_prompt, risk_score, user_context):
    enrichment = base_prompt

    # High-risk user enrichment
    if user_context.risk_history_score > 50:
        enrichment += "\n\n[SECURITY CONTEXT] This user has a history of flagged requests. "
        enrichment += "Apply maximum scrutiny. Default to refusal for ambiguous requests."

    # Session-level enrichment
    if user_context.session_flag_count > 3:
        enrichment += "\n\n[SECURITY CONTEXT] Multiple flags in this session. "
        enrichment += "Require explicit confirmation for all code generation requests."

    # Time-based enrichment (attacks often happen outside business hours)
    if is_off_hours(user_context.timestamp):
        enrichment += "\n\n[SECURITY CONTEXT] Off-hours request. "
        enrichment += "Additional verification required for sensitive topics."

    return enrichment
```

### Layer 7: System Prompt Hardening
**Purpose:** Make the model itself resistant to manipulation.

*See System Prompt Hardening Template for full implementation.*

Additional hardening for this layer:
```markdown
## Additional System Instructions for Multi-Layer Defense

### Conversation State Awareness
You have access to the following conversation metadata:
- Risk score of this session: [RISK_SCORE]
- Number of previous flags: [FLAG_COUNT]
- User trust level: [TRUST_LEVEL]
- Time since last refusal: [TIME_SINCE_REFUSAL]

Use this information to maintain consistent safety posture. 
Do not lower your guard because of previous benign interactions.

### Cross-Reference Validation
Before generating code, validate:
1. Does the code match the user's stated intent?
2. Does the code only access resources the user has authority over?
3. Does the code contain any operations that could harm systems or data?
4. Does the code require privileges the user may not have?

If any check fails, refuse and explain what legitimate alternative you can provide.
```

### Layer 8: Model-Level Safety
**Purpose:** Ensure the base model and fine-tuning resist harmful requests.

```yaml
model_safety:
  base_model_selection:
    criteria:
      - "Safety training aligned with your organization's values"
      - "Refusal behavior consistent across languages and encodings"
      - "Robustness to multi-turn attacks in benchmark evaluations"

  fine_tuning:
    dataset:
      - "Curated refusal examples for your specific use cases"
      - "Adversarial examples from red team exercises"
      - "Domain-specific benign examples to reduce false positives"

    techniques:
      - "RLHF with safety-weighted reward model"
      - "Constitutional AI with organization-specific constitution"
      - "Adversarial training with generated attack variations"

  evaluation:
    red_team_benchmarks:
      - "Direct injection success rate < 1%"
      - "Jailbreak success rate < 2%"
      - "Multi-turn exploitation success rate < 5%"
      - "False positive rate on legitimate requests < 3%"
```

### Layer 9-12: Output Validation
**Purpose:** Catch harmful content that slips through previous layers.

*See Output Constraint & Filtering Template for full implementation.*

Additional multi-layer checks:
```python
# Cross-layer validation
def validate_across_layers(output, input_analysis, user_context):
    # Check if output contains content that wasn't in the input
    # (indicates potential indirect injection or hallucination)
    novel_content = extract_novel_content(output, input_analysis)
    if novel_content and input_analysis.risk_score > 50:
        return {
            "action": "FLAG",
            "reason": "High-risk input produced novel content not grounded in input"
        }

    # Check for consistency across model runs
    if user_context.sensitivity_level == "HIGH":
        # Run the same input through a second, simpler model
        # and compare outputs for divergence
        second_output = simple_model.generate(input_analysis.normalized_input)
        divergence = calculate_semantic_divergence(output, second_output)
        if divergence > 0.5:
            return {
                "action": "FLAG",
                "reason": "High output divergence between models — potential injection"
            }

    return {"action": "PROCEED"}
```

### Layer 13-15: Monitoring & Response
**Purpose:** Detect and respond to attacks in real-time.

```yaml
monitoring:
  siem_integration:
    log_format: "CEF (Common Event Format)"
    events:
      - "BLOCK: injection_attempt_detected"
      - "BLOCK: harmful_content_generated"
      - "FLAG: suspicious_input_pattern"
      - "FLAG: output_content_mismatch"
      - "WARN: multi_turn_escalation_detected"
      - "INFO: user_rate_limit_exceeded"

    fields:
      - "user_id"
      - "timestamp"
      - "risk_score"
      - "matched_patterns"
      - "input_hash (for privacy)"
      - "output_hash (for privacy)"
      - "session_id"
      - "action_taken"

  soc_alerting:
    critical_threshold:
      - "3+ BLOCK events from same user in 1 hour"
      - "1+ BLOCK event with CRITICAL severity"
      - "5+ FLAG events from same user in 1 hour"
      - "New attack pattern detected (not in pattern library)"

    alert_channels:
      - "PagerDuty"
      - "Slack #security-alerts"
      - "Email to security-team@company.com"

  auto_response:
    immediate_actions:
      - "Temporarily block user after 3 BLOCK events"
      - "Require re-authentication after 5 FLAG events"
      - "Quarantine session logs for forensic analysis"

    escalation:
      - "L1: Automated blocking (0-5 minutes)"
      - "L2: SOC analyst review (5-30 minutes)"
      - "L3: Security team investigation (30 minutes - 24 hours)"
      - "L4: Incident response team (24+ hours or confirmed breach)"
```

---

## Defense Matrix: Red Team Attack vs. Blue Team Layer

| Attack Vector | Primary Defense Layer | Secondary Layers | Detection Method |
|---------------|----------------------|------------------|------------------|
| Direct Injection | Layer 7 (System Prompt) | L4, L5, L9 | Pattern match + canary check |
| Indirect Injection | Layer 3 (Normalization) | L9, L10, L12 | Hidden content detection |
| Jailbreak / Roleplay | Layer 7 (System Prompt) | L5, L8, L10 | Semantic classification |
| Obfuscation | Layer 3 (Normalization) | L4, L5 | Encoding detection + homoglyph neutralization |
| Multi-Turn Exploitation | Layer 6 (Context Enrichment) | L5, L13 | Conversation state analysis |
| Code Generation Bypass | Layer 11 (Code Safety Scan) | L7, L8, L10 | Static analysis + intent mismatch |
| Token Smuggling | Layer 3 (Normalization) | L4, L9 | Zero-width character detection |
| Context Window Stuffing | Layer 1 (Rate Limit) | L3, L7 | Input length + entropy analysis |

---

*This defense-in-depth architecture ensures that no single layer is the sole point of failure. Attackers must bypass multiple independent controls to succeed.*
