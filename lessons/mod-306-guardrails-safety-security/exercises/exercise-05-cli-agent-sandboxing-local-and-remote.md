# exercise-05: CLI Agent Sandboxing, Local and Remote

**Estimated effort:** 3 hours

## Objective

Design the **sandbox architecture** for a code-executing CLI agent in two topologies — a developer's local machine and a remote service in your infrastructure — so that a hijacked or over-eager agent can neither destroy data nor exfiltrate secrets. By the end you will have a containment design for every sandbox dimension and a clear account of why the local topology is harder and what you do about it.

## Background

This exercise covers material from:

- [Chapter 5 — Sandboxing Local and Remote CLI Agents](../05-sandboxing-cli-agents.md)
- [Chapter 3 — Least Privilege, Sandboxing, and Human Approval](../03-least-privilege-and-approval.md)

The agent: a **coding assistant** that can run shell commands, edit files, run tests, and fetch packages, on behalf of a developer. It must remain useful (install deps, run the test suite, format code) while being unable to do the two things that end careers: wipe data or leak credentials. The deliverable is a sandbox design document with a topology diagram per environment — design plus config sketches, not a running sandbox.

## Prerequisites

- The sandbox-dimensions checklist from Chapter 5.
- Basic familiarity with containers, filesystem mounts, and network egress control.

## Tasks

### 1. Threat statement

- Write the two concrete worst-case actions for this agent (one destructive, one exfiltration), each as a specific command, and state which sandbox dimension must stop each.

### 2. Design the remote sandbox

- Specify each dimension: filesystem (base image, writable scratch, no secret mounts), credentials (broker-injected, none resident), **egress** (default-deny allowlist — name the legitimate hosts), process/syscall hardening (non-root, dropped caps, seccomp, no-new-privileges), resource limits, and ephemeral lifetime.
- Draw the topology with the egress proxy.

### 3. Design the local sandbox

- The agent runs on a machine full of secrets. Specify: run in a container/dev-VM with only the project dir mounted, a **command allowlist**, mandatory approval for destructive verbs (`rm`, `git push --force`, `kubectl delete`) and anything touching `~/.ssh`, `~/.aws`, `~/.config`, and egress control even locally.
- Explain explicitly why you *cannot* achieve the same isolation as remote, and what residual risk remains.

### 4. Egress policy

- Write the egress allowlist for both topologies and show how it blocks the exfiltration command from task 1 even if the agent is fully hijacked.

### 5. Defense-in-depth argument

- Show that escaping any single boundary still lands inside another (e.g. egress leaks but no resident credentials exist to steal).

## Starter guidance

Remote topology:

```text
   ┌──────────────────────────────────────────────┐
   │  ephemeral container / microVM                │
   │   coding agent  |  fs: ro base + /work scratch│
   │                 |  creds: none resident       │
   │                 |  user: non-root, seccomp,   │
   │                 |        --cap-drop=ALL        │
   │       egress (default-deny) │                 │
   └─────────────────────────────┼─────────────────┘
                                 ▼
                          ┌─────────────┐  allow: pkg registry, internal git
                          │ egress proxy│  deny:  everything else (logged)
                          └─────────────┘
```

Hardening config sketch (illustrative):

```text
# container runtime flags
--read-only                       # ro root fs
--tmpfs /work:rw,size=512m        # single writable scratch
--user 10001:10001                # non-root
--cap-drop=ALL                    # no Linux capabilities
--security-opt=no-new-privileges  # no setuid escalation
--security-opt=seccomp=agent.json # restricted syscalls
--network=egress-controlled       # default-deny egress via proxy
--memory=2g --cpus=2 --pids-limit=256
```

Local command-policy sketch:

```text
allow (auto):   npm ci | pytest | ruff format | git status | git diff
approve (human): rm * | git push --force | kubectl delete | anything reading ~/.ssh ~/.aws
deny (always):  curl|wget to non-allowlisted host | reading credential caches for output
```

## Acceptance criteria

You can demonstrate that:

- The threat statement names a concrete destructive and exfiltration command and the dimension that stops each.
- The remote sandbox specifies all dimensions, including a named egress allowlist and an egress proxy.
- The local sandbox uses a container/VM, command allowlist, approval on destructive/credential-touching commands, and egress control, with the residual-risk gap stated honestly.
- The egress policy provably blocks the exfiltration command even under full hijack.
- The defense-in-depth argument shows no single boundary is load-bearing alone.

## Reflection

In `NOTES.md`:

1. What useful capability did you have to give up (or gate behind approval) to close the destructive path? Was it worth it?
2. In the local topology, which residual risk worries you most, and what operational control (not technical) reduces it?
3. How would you detect — from logs — that a sandboxed agent attempted an exfiltration?

## Stretch goals

- Add a **post-run diff review**: the agent's filesystem changes are surfaced as a diff for human approval before they touch the real project.
- Design the egress proxy to do TLS-aware host allowlisting and per-destination rate limits.
- Write adversarial tests (Chapter 6) asserting that a planted exfiltration command results in `EGRESS_BLOCKED`, not a refusal.
