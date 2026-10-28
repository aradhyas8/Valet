# Competitive check (step 1)

Date: 2026-10-28. Sources are mostly vendor READMEs, docs and launch posts. None of the security claims below were independently tested.

## Bottom line

The secret-swapping proxy (agent sends `{{valet:stripe}}`, proxy injects the real key only for allowed hosts) is **not novel**. At least five open projects ship it today. Valet cannot lead with it.

What still looks open is **action-level policy plus a human approval step that shows exactly what will happen** (the email text, the recipient, the refund amount), in a self-hosted, agent-agnostic package. Competitors match on host/path/method. Few, if any, understand the action, and approval workflows are in-progress PRs or on roadmaps.

## Open-source credential proxies

| Project | What it does | Approvals | Policy depth |
|---|---|---|---|
| [Infisical Agent Vault](https://github.com/Infisical/agent-vault) (MIT, research preview) | HTTP proxy + encrypted vault. Strips agent-supplied creds, injects real ones. Targets Claude Code, OpenClaw, Hermes by name. | Approval for *new service access* (proposal URL). Per-request approval is an open PR (#434, Sep 2026). | Host/service level |
| [Aegis](https://github.com/getaegis/aegis) | Local transparent proxy with domain guard | None documented; webhook alerts on blocks | YAML: method, path, rate limit, time of day; body scan for leaked creds |
| [OneCLI](https://onecli.sh) (YC) | Credential gateway; default credential layer in NanoClaw | On roadmap (host/path/method/agent) | Host/path |
| [Keys on the Wire](https://github.com/inflightsec/keys-on-the-wire) (Apache-2.0) | Loopback HTTPS proxy, JIT fetch from vault; OAuth2, SigV4, HMAC | None found | Fails closed outside a key's scope |
| [agent-vault-proxy](https://github.com/inflightsec/agent-vault-proxy) (MIT) | Same idea, backed by Bitwarden Secrets Manager | None found | Scope-based |
| AgentSecrets | Localhost proxy reading OS keychain | Unknown | Unknown |

## Commercial

- **1Password.** [Credential Broker](https://1password.com/blog/introducing-1password-credential-broker) (announced June 2026; GitHub Actions first, AI agents planned). [1Password for Claude](https://www.digitalapplied.com/blog/1password-claude-secure-credential-agentic-browsing-2026) (July 2026) does browser logins with biometric per-task approval. Late-October reports describe a "1Password for Agents" product for personal agents. This overlaps directly with Valet's v2 "logged-in browser sessions" idea and with the approval story. Closed source, tied to a 1Password account.
- **Arcade.dev.** Tool catalog + auth. Engine can be self-hosted. Its "human in the loop" is mainly OAuth consent; a Slack approval example exists as a template, not a core feature (third-party reviews say per-call approval isn't documented).
- **Composio.** Managed cloud tool catalog. No self-host evidence found.

## Where Valet can still differentiate

1. **Semantic, per-action policy.** `send_to: *@mycompany.com`, `refund.max_amount: 50`, `push_to_main: deny`. Requires parsing requests for known APIs (or MCP tools), not just matching hosts. Aegis is closest but has no approvals.
2. **Approval UX that shows the payload in human terms** on the phone, and approves *intent* rather than a body hash (a known pain point: one fork's issue complains each retry costs a new approval).
3. **One policy across MCP tools and the proxy**, so the same rule applies however the agent calls the API.
4. **Open, self-hosted, no account** vs 1Password.

Risk: Infisical is a well-funded secrets company with the same target agents and an approval PR already open. The window is months, not years.

## Options

- **A. Narrow and go deep:** build Valet as the policy + approval layer, with the proxy as plumbing. Lead with the "approve the email on your phone" demo.
- **B. Build on top of an existing proxy:** contribute approvals/semantic policy to Agent Vault or Aegis, or ship Valet as a policy/approval add-on for them. Less to build, less to own.
- **C. Pick a sharper niche:** e.g. only personal always-on agents (OpenClaw/Hermes) with a polished phone approval app.

Recommendation: **A**, but build the proxy minimally (or reuse an Apache/MIT one) and spend the effort on policy + approvals.
