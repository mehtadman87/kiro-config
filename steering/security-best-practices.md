---
title: Security Best Practices
inclusion: always
---

# Security Best Practices

## Code Security
- Never hardcode secrets, API keys, or passwords
- Use environment variables for configuration
- Validate all user inputs
- Use parameterized queries to prevent SQL injection
- Implement proper authentication and authorization

## Dependency Management
- Keep dependencies updated
- Use dependency scanning tools
- Review third-party packages before adding
- Use lock files (package-lock.json, poetry.lock)
- Remove unused dependencies

## Data Protection
- Encrypt sensitive data at rest and in transit
- Use HTTPS for all web communications
- Implement proper session management
- Use secure headers (HSTS, CSP, etc.)
- Follow OWASP guidelines

## Infrastructure Security
- Use least privilege principle for IAM
- Enable logging and monitoring
- Use network segmentation
- Implement proper backup strategies
- Regular security audits and penetration testing

## Development Practices
- Use static code analysis tools
- Implement security testing in CI/CD
- Code reviews for security issues
- Security training for developers
- Incident response procedures

## LLM & Agentic AI Security (Research-Validated)

These patterns are grounded in production agentic AI security research (2025-2026).

### Guardrails: Layered Protection Model

Effective guardrails operate at three layers, and all three are required for production systems:

**Layer 1: Input Guardrails (Pre-Processing)**
- Input classification: detect and block prompt injection, jailbreak attempts, and out-of-scope requests before they reach the LLM
- Input sanitization: strip or escape potentially malicious content from user inputs and retrieved documents
- Schema validation: enforce structured input formats where applicable
- Rate limiting: prevent abuse through excessive or rapid requests

**Layer 2: Runtime Guardrails (During Processing)**
- Tool permission enforcement: restrict which tools an agent can call based on the current task and user permissions
- Budget controls: set hard limits on token usage, API calls, and compute time per request
- Scope constraints: prevent agents from accessing resources or performing actions outside their designated domain
- Intermediate output validation: check agent reasoning at each step for policy violations

**Layer 3: Output Guardrails (Post-Processing)**
- Content filtering: scan outputs for harmful, biased, or policy-violating content
- Factual grounding checks: verify claims against retrieved sources
- Format validation: ensure outputs conform to expected schemas
- PII detection: scan for and redact personally identifiable information before delivery

Source: "Current state of LLM Risks and AI Guardrails" [arxiv.org/html/2406.12934]; "Auto-Tuning Safety Guardrails for Black-Box LLMs" (2025) [arxiv.org/html/2512.15782]

### Guardrail Implementation Strategies

- **Use dedicated guardrail models.** Deploy smaller, fine-tuned classifier models specifically for safety checks rather than relying on the primary LLM to self-police.
- **Hybrid neural-symbolic systems.** Combine LLM-based classifiers with rule-based checks. Rules catch known patterns reliably; neural classifiers catch novel threats.
- **Adaptive guardrails.** Adjust strictness based on task risk level, user trust level, and confidence scores.

### Guardrail Anti-Patterns
- Relying solely on system prompt instructions for safety (easily bypassed)
- Hand-tuning guardrails without systematic testing (brittle and hard to reproduce)
- Applying uniform guardrail strictness regardless of task risk
- Skipping output validation because "the model is smart enough"

### Prompt Injection Defense (Defense-in-Depth)

Prompt injection is ranked #1 on the OWASP Top 10 for LLM Applications (2023, 2025 updates). Researchers have demonstrated 100% evasion success against some commercial prompt shields; no single defense is sufficient.

**Layer 1: Input Classification**
- Deploy a fine-tuned classifier model to detect prompt injection attempts before they reach the primary LLM
- Combine with heuristic pattern matching for known attack signatures
- Maintain and update a denylist of known injection patterns

**Layer 2: Prompt Design (Privilege Separation)**
- Separate system instructions from user input using clear delimiters and namespaces
- Use XML tags or other structural markers to delineate trusted (system) vs untrusted (user) content
- Never concatenate user input directly into system prompts without sanitization
- Implement instruction hierarchy: system prompt instructions take precedence over user input

**Layer 3: Tool Permission Minimization**
- Only expose tools that are needed for the current task
- Implement per-tool authorization checks
- Use read-only tool variants where write access isn't required

**Layer 4: Output Filtering**
- Scan outputs for signs that injected instructions were followed
- Check for data exfiltration patterns (outputs containing system prompt content, internal data, or credentials)
- Validate output format matches expected schema

**Layer 5: Runtime Monitoring**
- Track anomalous patterns: unusual tool call sequences, unexpected output formats, sudden behavior changes
- Implement rate limiting per user/session
- Alert on potential injection attempts for human review
- Maintain audit logs of all agent interactions

### Agentic AI Security Practices
- **Least privilege:** Agents should have the minimum permissions needed for their task. Never give an agent admin access "just in case."
- **Sandboxing:** Execute agent-generated code in isolated environments (containers, VMs, sandboxed runtimes)
- **Secret management:** Never pass secrets through LLM context. Use environment variables or secret managers accessed by tools, not by the LLM directly.
- **Data classification:** Tag data by sensitivity level; restrict agent access to sensitive data based on task requirements and user authorization
- **Regular red-teaming:** Periodically test agent systems with adversarial inputs to identify vulnerabilities before attackers do