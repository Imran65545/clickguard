# ClickGuard

**Context-aware AI security for suspicious command execution.**
*Understand the action. Detect the deception.*

> **Status: idea stage.** ClickGuard is a proposed system and a planned prototype, built for the CyberForge hackathon. There is no implementation or test result yet. Everything below describes the plan.

**Team CTRL ALT ELITE:** Choudhary Imran, Mohammed Tabish

## The problem

ClickFix-style attacks make the user run the attack themselves:

1. A deceptive verification or error page persuades the user to follow instructions.
2. A suspicious command is placed on the clipboard or shown for copying.
3. The user runs it through a legitimate Windows utility.

Legitimate system tools can be abused to run attacker-supplied commands, so behavioural context matters. Microsoft notes that this kind of campaign "could get past conventional and automated security solutions" ([Microsoft Security Blog](https://www.microsoft.com/en-us/security/blog/2025/08/21/think-before-you-clickfix-analyzing-the-clickfix-social-engineering-technique)).

## The idea

ClickGuard correlates clipboard activity, foreground application context, shell command details and process behaviour on Windows, and judges whether the action makes sense in context.

| Example | Planned outcome |
| --- | --- |
| A PDF reader starts a shell with suspicious arguments | High-risk alert, attempted response under policy |
| A page copies a suspicious command, then a shell runs shortly after | Correlation raises risk |
| A developer deliberately runs trusted build tools | Context supports allowing it |

Signed software is not treated as automatically safe, and there is no unconditional allowlist.

## Proposed architecture

1. **Activity:** clipboard changes, foreground window, new processes, parent-child links, matched by timestamp. Individual browser clicks are not directly visible.
2. **Windows agent:** event-driven, using the Win32 clipboard listener, foreground-window APIs, ETW or WMI process events, and psutil.
3. **Local engine:** rules and risk scoring decide first and work offline. Only ambiguous cases get AI analysis.
4. **Response:** allow, warn, log, attempt suspend, resume, or terminate when authorized, each with a plain-language reason.

Backend (planned): Node.js, MongoDB, Socket.IO, Next.js dashboard, LLM API. It handles telemetry, AI investigation and the dashboard, and is not the only decision point.

### Hybrid engine (design targets, not measured)

| Tier | Job | Target |
| --- | --- | --- |
| 0 Trusted context | Known workflows lower risk, no unconditional bypass | under 5 ms |
| 1 Rules | Predefined high-risk patterns, handled locally | under 10 ms |
| 2 Risk score | Ancestry, shell arguments, clipboard match, hidden or encoded execution, timing | under 50 ms |
| 3 AI review | Ambiguous cases only, structured verdict and reason | 1 to 3 s |

The AI cannot overturn a high-confidence rule block. Commands and clipboard text are treated as untrusted input, and model output is schema-validated.

## Limits we are upfront about

- The MVP aims at **detection and post-launch response**. A fast process may run before the agent can act, and suspension may fail, so we do not claim prevention.
- Guaranteed pre-execution blocking needs stronger Windows enforcement (Phase 2) and is not provided by Python, psutil, WMI or ETW alone.
- The Windows agent sees OS-level events in any browser, but not the exact click or URL. A browser extension is on the roadmap.
- Event availability, permissions and delivery delays must be validated.

## Planned validation (no results yet)

Detection rate in labeled safe simulations, false positives in normal workflows, event and alert latency, response success vs failure, CPU and memory, AI request count and cost, offline behaviour. All tests will use harmless commands in an authorized test environment.

## Roadmap

1. **MVP:** Windows telemetry, clipboard and process correlation, local rules, AI triage, dashboard.
2. **Reliable protection:** evaluate stronger Windows enforcement for pre-execution policy.
3. **Expansion:** browser extension, enterprise policy, separately researched mobile support.

## Sources

- [Microsoft Security Blog: Think before you Click(Fix)](https://www.microsoft.com/en-us/security/blog/2025/08/21/think-before-you-clickfix-analyzing-the-clickfix-social-engineering-technique)
- [Opera: Paste Protect](https://press.opera.com/?p=3955)
- Microsoft Learn: [AddClipboardFormatListener](https://learn.microsoft.com/en-us/windows/win32/api/winuser/nf-winuser-addclipboardformatlistener), [Event Tracing for Windows](https://learn.microsoft.com/en-us/windows/win32/etw/about-event-tracing), [PsSetCreateProcessNotifyRoutineEx](https://learn.microsoft.com/en-us/windows-hardware/drivers/ddi/ntddk/nf-ntddk-pssetcreateprocessnotifyroutineex)
