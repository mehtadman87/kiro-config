---
title: Email Drafting Style Guide
inclusion: manual
---

# Email Drafting Style Guide

When asked to draft an email, follow these rules precisely. This style reflects how the SA communicates with customers and internal stakeholders.

## Subject Line Format

```
Subject: [EXTERNAL] AWS/{Customer Name} - {Subject/Topic}
```

## Salutation

Use first name only, followed by a comma and a line break. No "Hi" or "Hello" prefix.

```
John,
```

## Structure and Flow

- Open directly with the topic — no pleasantries or preamble
- Use a bold section heading (no colon) immediately after the salutation to introduce the first topic
- Transition between topics with prose paragraphs, not headers, unless a new major section warrants one
- Use a "Recommendations" or "Path for [Customer]" section near the end to summarize actionable next steps
- Close with "Happy to walk through any of these in more detail on a call." before the sign-off
- Sign off with "Thanks," followed by a blank line (no name — the email signature handles that)

## Tone and Voice

- Direct, technical, and authoritative — write as a trusted advisor, not a vendor
- No filler phrases: avoid "Great question," "I hope this helps," "Please don't hesitate," "Feel free to"
- No exclamation points
- No em dashes (—) in body prose; use commas or restructure the sentence instead
- Confident declarative sentences; avoid hedging language like "might want to consider" unless genuinely uncertain

## Bullet Points and Lists

- Use bullet points sparingly — prefer prose for explanations
- When bullets are used, each bullet ends with a period
- Numbered lists are acceptable for sequential steps or ranked options
- For recommendation sections with multiple options, introduce each with "The first is..." / "The second is..." as prose rather than a bulleted list

## Hyphens

Do not use hyphens (-) as list markers or bullet substitutes. Use actual bullet points or numbered lists only.

## Referencing Documentation and Links

- Use inline numbered reference markers in square brackets: `[1]`, `[2]`, `[3]`
- Place a "References:" section at the bottom of the email with the full URLs
- When quoting documentation directly, use double quotation marks and include the reference marker immediately after the closing quote
- Direct quotes from AWS documentation are preferred over paraphrasing when precision matters
- Example: "Customizations made by a delegated administrator account will be saved independently from customizations made by member accounts" [2].

## Explaining Technical Concepts

- State what the feature/behavior IS before explaining what it means for the customer
- Follow the pattern: [what it is] → [why it matters in this context] → [what the customer should do]
- When a limitation exists, acknowledge it plainly, then pivot immediately to what CAN be done
- If submitting a PFR on the customer's behalf, state it directly: "I have submitted a feature request on your behalf for this gap."

## Recommendations Section

- Title it "Recommendations" or "Path for [CustomerName]" depending on context
- Organize by time horizon when applicable: Near-term / Medium-term / Strategic
- Each recommendation should be one to three sentences: what it is, why it helps, any caveat
- Do not use sub-bullets inside recommendation items — keep each item as a prose paragraph or a single sentence

## References Section

Format exactly as:

```
References:
[1] https://...
[2] https://...
[3] https://...
```

No link text, no markdown hyperlinks — just the numbered label and the raw URL.

## What to Avoid

- Do not bold random phrases for emphasis mid-sentence
- Do not use "please note that" or "it is important to note"
- Do not use passive voice when active voice is clearer
- Do not use "leverage" as a verb — use "use" instead
- Do not use "utilize" — use "use"
- Do not start sentences with "So," or "Now,"
- Do not use "deep dive" as a noun or verb
- Do not use "at the end of the day"
- Do not use "moving forward"
- Do not use "circle back"
- Do not use "synergy" or "synergize"

## Example Structure

```
Subject: [EXTERNAL] AWS/{Customer} - {Topic}

{FirstName},

{Bold Section Heading — no colon}
{Opening paragraph explaining the core concept or answer directly.}

{Continuation paragraph with technical detail, quoting docs where relevant [1].}

{Transition paragraph connecting to recommendations.}

Recommendations
{Near-term: ...}
{Medium-term: ...}
{Strategic: ...}

Happy to walk through any of these in more detail on a call.

References:
[1] https://...
[2] https://...

Thanks,
```
