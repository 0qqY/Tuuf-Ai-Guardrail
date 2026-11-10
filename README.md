# Tuuf Ai Guardrail

A self-hosted security layer for LLM chatbots. It sits between your application and the language model behind it, screens every user message and every model answer, and runs fully offline.

**Status: pre-release, in pilot preparation.** This page describes what the product does today. It is not generally available yet, and no performance figures are published here; they will come with a technical report that states every test set and its size.

## What it does

- **Screens both directions.** User messages are checked before they reach the model; model answers are checked before they reach the user.
- **Prompt injection and harmful requests.** Detection of prompt injection, jailbreak attempts and harmful requests. Text is normalised first, and Base64-encoded or invisible-character obfuscation is decoded and checked too.
- **Personal data.** Turkish national ID numbers (checksum verified), phone numbers and e-mail addresses can be masked, blocked or allowed. Masking formats are configurable (whole value as a tag, or only some digits visible). The text sent on to the model and stored in the log is the masked text.
- **System prompt leakage.** Answers are checked for leaks of the system prompt, using a canary token and fragment matching.
- **Custom rules in plain language.** An administrator describes in Turkish or English what the chatbot must not talk about, or may only talk about. The administrator's own LLM drafts the rule; the administrator reviews, tests and publishes it as part of a versioned policy.
- **Conversation drift.** With a session identifier, each message adds to a risk score for the conversation. First the message is flagged for review, then the conversation is locked until the user returns to the allowed topic.
- **Distress routing (Turkish).** Messages that suggest self-harm are not sent to the model; the user receives a supportive message pointing to emergency help. The message text is not stored. This is a referral, not a crisis service.
- **Administration.** Versioned policy with rollback, an audit log, roles (Global Admin, Analyst, Auditor), username and password or LDAP sign-in, and a web panel.
- **SIEM.** Decisions and audit events are exported to Splunk (HTTP Event Collector) or Elasticsearch, through a persistent queue that retries when the SIEM is unreachable. Message text is not sent unless enabled.

## Deployment

- One container image for Linux with an NVIDIA GPU (8 GB or more). No internet access is needed at run time; models are inside the image.
- HTTPS by default (the installer creates a certificate; you can supply your own). The container runs with a read-only file system, without Linux capabilities and without root.
- Works with any OpenAI-compatible language model, in your network or in the cloud.

## Integration

Two REST endpoints, authenticated with an API key:

- `POST /check`: your application sends a message or an answer and receives a decision (allow or block), the reasons and the masked text.
- `POST /chat`: your application sends the user's message; the service checks it, calls your language model, checks the answer and returns it.

Optional fields (`app`, `user_id`, `session_id`) travel into the audit and SIEM events.

## Known limitations

- Detection of prompt injection in English is weaker than in Turkish on public third-party benchmarks. Improving it is the main work in progress.
- Distress routing covers Turkish only.
- No OpenAI-compatible proxy endpoint yet; applications call `/check` or `/chat`.
- No two-factor sign-in yet.
- It is a screening layer, not a replacement for access control in your application.

## Contact

Orhan Öztaş: 
Mail : orhanoztasbir@gmail.com
Linkedin : https://www.linkedin.com/in/orhanoztas

*This project is not affiliated with or endorsed by OWASP, Anthropic, OpenAI or any model provider named by its users.*
