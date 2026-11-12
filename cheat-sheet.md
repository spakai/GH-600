# GH-600 — GitHub Agentic AI Developer

**Exam date: 27 November 2026**
**Purpose:** fast revision of agentic development concepts and GitHub Copilot workflows.

> This workspace identifies GH-600 as “GitHub Agentic AI Developer,” but does not include an official exam blueprint. Use this as a topic guide, and check the current GitHub Learn exam page for the assessed domains, format, and feature availability.

## 1. Agentic AI: the core loop

An AI agent is more than a chat interface: it can pursue a goal over multiple steps, use tools, observe results, and adjust its next action.

**Goal → plan → act → observe → reflect/iterate → verify → human review**

- **Model:** interprets the request, reasons, and selects next actions.
- **Instructions/context:** define the goal, constraints, repository conventions, and relevant facts.
- **Tools:** let the agent read or change things outside the model, such as files, terminals, APIs, or issues.
- **Orchestration/runtime:** controls the agent loop, available tools, permissions, and stop conditions.
- **Human:** supplies intent, grants appropriate access, reviews consequential actions, and accepts or rejects the result.

**Remember:** the model proposes; tools perform; the runtime enforces boundaries; a person remains accountable.

## 2. Copilot surfaces — know the difference

| Surface | Best fit | Typical workflow |
|---|---|---|
| **Copilot Chat** | Ask questions, explain code, brainstorm, or get a focused suggestion | You provide context and decide what to apply |
| **IDE agent mode** | Work interactively across files and run tools/tests | Agent edits locally, reports actions, and iterates with you |
| **Copilot cloud agent** | Delegate a repository task asynchronously | Agent researches, changes code in an isolated environment, and opens a PR for review |
| **Copilot CLI** | Agentic assistance from a terminal | Work with repository context and terminal-oriented tasks |

Pick the least autonomous surface that can complete the task. Delegation does not remove the need to inspect the diff, test, and review the PR.

## 3. Give agents a task they can finish

A strong request states:

1. **Outcome:** what should be true when finished?
2. **Context:** relevant files, issue, behavior, or constraints.
3. **Scope:** what is in and out of bounds?
4. **Acceptance criteria:** observable requirements, including edge cases.
5. **Validation:** tests, build, lint, or manual checks to run.
6. **Reporting:** ask for a concise summary, changed files, and test results.

**Reusable prompt**

> Implement **[outcome]** in **[scope]**. Follow **[relevant convention/context]**. Do not change **[out-of-scope behavior]**. Acceptance criteria: **[checklist]**. Run **[validation commands]** and report the results, any limitations, and the files changed.

For ambiguous, risky, or broad work: ask for a plan first, narrow the task, and approve consequential tool actions. For small, well-defined changes: ask for implementation and verification directly.

## 4. Context and customization

Use the right mechanism for the kind of guidance:

- **Prompt:** task-specific goal and acceptance criteria.
- **Repository instructions:** durable conventions and project-wide guidance.
- **Path-specific instructions:** rules that apply only to matching files.
- **Prompt files:** reusable task templates that a developer invokes.
- **Custom agents:** focused role/persona and tool access for a class of work.
- **Skills:** reusable task knowledge and workflows, typically discovered when relevant.
- **MCP:** connections to external tools and data.

More context is not automatically better. Supply the smallest relevant, trustworthy context; point to authoritative files and identify which constraints matter. Keep instructions specific, non-conflicting, and testable.

### MCP essentials

- **MCP (Model Context Protocol)** is an open standard for connecting AI applications with external context and capabilities.
- A **host** is the AI application; an **MCP client** manages a connection; an **MCP server** exposes capabilities.
- Know the capability categories: **tools** (actions), **resources** (context/data), and **prompts** (reusable prompt templates).
- MCP extends the agent's available tools/data; it does not replace task instructions or grant safe judgment.
- Before enabling a server, verify its source, requested permissions, configuration, and data handling. Use least privilege and avoid exposing secrets.

## 5. Safe and reliable agent use

- Treat repository content, issues, web pages, and tool output as **untrusted input**. They may contain instructions that attempt to redirect the agent.
- Do not let untrusted content override system/developer policy, user intent, or repository security requirements.
- Grant only the permissions needed for the task; require approval for sensitive or destructive actions.
- Never paste credentials into prompts or commit them. Use approved secret stores and environment mechanisms.
- Review generated code for insecure patterns, dependency changes, license implications, and unintended scope.
- Run the relevant tests and inspect failures; a confident explanation is not proof of correctness.
- Keep human review for external side effects (publishing, deployments, permissions, data changes) and merge decisions.
- Agent output can be plausible but wrong: verify facts against source code, tests, and trusted documentation.

**Common failure modes:** vague goals, missing context, oversized tasks, conflicting instructions, untrusted prompt injection, excessive permissions, unverified edits, and assuming that passing one test proves overall correctness.

## 6. Human-in-the-loop delivery

For a delegated code change, the dependable workflow is:

1. Create a well-scoped issue/task with acceptance criteria.
2. Delegate to the cloud agent or work interactively in the IDE.
3. Inspect the agent's plan and any requested permissions.
4. Review the resulting diff for correctness, scope, maintainability, and security.
5. Run tests/CI and investigate failures rather than accepting a success-shaped summary.
6. Request changes or follow up with a precise prompt if needed.
7. Approve and merge only after normal repository review requirements are met.

**A PR is a review boundary, not a guarantee.** Confirm the agent changed the intended branch/files and that checks correspond to the final commit.

## 7. Evaluation checklist

Judge an agentic solution on:

- **Task success:** does it satisfy every acceptance criterion?
- **Correctness:** do behavior and edge cases match the requirement?
- **Verification:** are tests appropriate, relevant, and passing?
- **Security/privacy:** were permissions and data exposure minimized?
- **Quality:** is the change maintainable and limited to the requested scope?
- **Transparency:** can a reviewer understand what the agent did and why?
- **Human control:** were approvals and review retained where needed?

For a repeated agent workflow, prefer measurable checks and repeatable tests over subjective “looks good” judgments.

## 8. Quick recall

- **Agent vs. chat:** an agent can take multi-step actions using tools; chat primarily responds to a prompt.
- **MCP vs. instructions:** MCP connects capabilities/context; instructions guide behavior.
- **Tool vs. resource:** a tool performs an action; a resource provides data/context.
- **IDE agent vs. cloud agent:** interactive local workflow vs. delegated repository work in a cloud environment that can produce a PR.
- **Prompt vs. durable instructions:** one task's request vs. reusable project guidance.
- **More autonomy requires:** clearer scope, stronger permission boundaries, and more deliberate review—not less.
- **Completion means:** acceptance criteria met and verified, not merely code generated.

## 9. Exam-focused reference

### Official exam map

| Domain | Weight |
|---|---:|
| Prepare agent architecture and SDLC processes | 15–20% |
| Implement tool use and environment interaction | 20–25% |
| Manage memory, state, and execution | 10–15% |
| Perform evaluation, error analysis, and tuning | 15–20% |
| Orchestrate multi-agent coordination | 15–20% |
| Implement guardrails and accountability | 10–15% |

Source: [Microsoft Learn study guide](https://learn.microsoft.com/en-us/credentials/certifications/resources/study-guides/gh-600). The scenarios below are study recommendations, not additional official exam objectives.

**Task contract:** objective → inputs → boundaries → deliverables → acceptance criteria → escalation condition.

For complex work, use inspect → plan → validate plan → execute → verify. For a small, reversible edit, use bounded execution with appropriate checks. Treat production release, destructive changes, secret changes, and merges as separate consequential actions requiring their applicable authorization and protections.

### Copilot CLI permission reminder

A narrow prompt does not narrow the credentials or tools available to the process. Use scoped credentials, grant only the needed tools, and verify CLI flags against the installed version.

```bash
copilot --allow-tool='shell(git:*)' --deny-tool='shell(git push)'
```

Allow/deny controls illustrate tool restrictions; they are not a complete sandbox or a substitute for credential scoping.

### GitHub Actions: validation and security

```yaml
name: validate-agent-pr
on:
  pull_request:
    types: [opened, synchronize, ready_for_review]

permissions:
  contents: read
  pull-requests: read

jobs:
  validate:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
        with:
          persist-credentials: false
      - name: Run tests
        run: npm ci && npm test
```

The snippet assumes the runner has a suitable Node version; configure the project's required version in a real workflow. `pull-requests: read` is for reading PR metadata; checkout and local tests generally need only `contents: read`.

| Syntax / clue | Meaning |
|---|---|
| `uses:` | Invoke an action |
| `run:` | Execute a shell command |
| `with:` | Supply action inputs |
| `opened` / `synchronize` | PR created / PR head updated |
| `ready_for_review` | Draft PR marked ready |
| `workflow_dispatch` | Explicit workflow dispatch |
| `workflow_run` | React to another workflow run |
| `workflow_call` | Reusable workflow called by another workflow |

Prefer `pull_request` when testing PR code. Do not execute untrusted PR code in a privileged `pull_request_target` context with secrets or write access. Read permissions limit token authority, but running PR code still executes untrusted code. Agent approval gates and repository Actions policies may also affect when CI runs.

To retain failure evidence, an upload step can use `if: always()`; this allows the step to run after earlier failures but does not create missing evidence. Sanitize files before uploading and set a suitable retention period.

### Durable run summary plus retained decision log

For validation on pull requests targeting `main`, write a concise Markdown summary to `$GITHUB_STEP_SUMMARY` and upload the detailed decision log as a workflow artifact:

```yaml
name: agent-validation
on:
  pull_request:
    branches: [main]

permissions:
  contents: read

jobs:
  validate:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
        with:
          persist-credentials: false

      - name: Run validation and capture decision log
        shell: bash
        run: |
          set -o pipefail
          ./scripts/validate-agents.sh 2>&1 | tee decision-log.md

      - name: Write workflow summary
        if: ${{ always() }}
        env:
          OUTCOME: ${{ job.status }}
        run: |
          {
            echo "## Agent validation"
            echo ""
            echo "Outcome: **$OUTCOME**"
            echo ""
            echo "See the workflow artifacts for the detailed decision log."
          } >> "$GITHUB_STEP_SUMMARY"

      - name: Upload decision log
        if: ${{ always() }}
        uses: actions/upload-artifact@v4
        with:
          name: agent-validation-decision-log
          path: decision-log.md
          retention-days: 30
          if-no-files-found: error
```

Replace the sample command with the platform team's validator. `tee` saves its output while preserving a failing exit code via `pipefail`; `always()` lets the summary and artifact steps run after validation failure. `$GITHUB_STEP_SUMMARY` appears on the workflow run page, while the artifact retains the detailed log for the configured period.

### Workflow completion and failure escalation

For “run after validation completes; create an issue only if validation fails”:

| Blank | Answer | Why |
|---|---|---|
| Trigger | `workflow_run` | React to another workflow |
| Permission | `issues: write` | Needed to create an issue |
| Condition | `${{ github.event.workflow_run.conclusion == 'failure' }}` | Reads the triggering workflow's result |

```yaml
name: escalate-agent-failure
on:
  workflow_run:
    workflows: [validate-copilot-pr]
    types: [completed]
permissions:
  contents: read
  issues: write
jobs:
  open-escalation:
    if: ${{ github.event.workflow_run.conclusion == 'failure' }}
    runs-on: ubuntu-latest
    steps:
      - name: Create escalation issue
        env:
          GH_TOKEN: ${{ secrets.GITHUB_TOKEN }}
          GH_REPO: ${{ github.repository }}
          RUN_URL: ${{ github.event.workflow_run.html_url }}
        run: |
          gh issue create --title "Agent validation failed" \
            --body "Inspect validation run: $RUN_URL"
```

`workflows` matches the upstream workflow's `name`, not its filename. The listener must exist on the default branch. `completed` includes successful and unsuccessful runs; the condition selects failure. Keep the listener limited to trusted operations: do not run upstream PR code or downloaded scripts. Prevent duplicate escalation issues in production.

| Operation | Typical permission |
|---|---|
| Read/check out repository files | `contents: read` |
| Push repository changes | `contents: write` |
| Read PR information / create or update PRs | `pull-requests: read` / `pull-requests: write` |
| Create or update issues | `issues: write` |
| Read Actions runs/artifacts / dispatch workflow runs | `actions: read` / `actions: write` |
| Publish packages | `packages: write` |
| Upload code-scanning results | `security-events: write` |
| Request an OIDC token | `id-token: write` |

When permissions are explicitly listed, unspecified permissions become `none`. OIDC token permission alone does not grant cloud access; the cloud trust policy must authorize it. Repository policies can restrict operations further.

**Don't confuse:** `actions: write` manages Actions operations; it does not grant issue creation. `failure()` checks the current execution context; use `github.event.workflow_run.conclusion` for the upstream run. A merged PR is a `closed` event plus a merged-state check—there is no `merged` activity type for `pull_request`.

### Copilot CLI hooks

| Hook | When it runs | Typical use |
|---|---|---|
| `sessionStart` | Session starts or resumes | Initialization or policy guidance |
| `userPromptSubmitted` | User submits a prompt | Audit logging |
| `preToolUse` | Before a tool executes | Validate arguments or deny prohibited actions |
| `postToolUse` | After a tool executes | Record results; cannot prevent the completed action |
| `sessionEnd` | Session ends | Cleanup or session audit summary |

**Recall:** start → guide; prompt → log; pre → prevent; post → record. Native repository hook configuration is stored in JSON files under `.github/hooks/`; ensure the scripts exist and paths resolve from the configured working directory.

For a `preToolUse` handler, the script's logic determines which calls it denies. Exiting successfully without a decision leaves normal permission checks in effect; it does not grant unrestricted access. An escalation message does not create an issue unless a separate authorized action does so. Regex command filters are not a complete security boundary; apply scoped credentials and execution controls too. Log prompts only with appropriate access, retention, and secret redaction.

## 10. Study plan: 12–27 November

Aim for **45–75 minutes per day**. Spend each session on active recall and a small hands-on exercise, not rereading alone.

| Date | Focus |
|---|---|
| **12 Nov** | Confirm official exam guide, objectives, exam format, and your weak areas |
| **13 Nov** | Agent loop, models vs. tools vs. orchestration; explain each from memory |
| **14 Nov** | Compare Copilot Chat, IDE agent mode, cloud agent, and CLI |
| **15 Nov** | Write three scoped prompts with explicit acceptance criteria |
| **16 Nov** | Instructions, prompt files, custom agents, and skills: choose the right mechanism |
| **17 Nov** | MCP architecture and tools/resources/prompts; trace one example connection |
| **18 Nov** | Security: least privilege, secret handling, untrusted content, prompt injection |
| **19 Nov** | Cloud-agent lifecycle: task, branch/work, PR, checks, human review |
| **20 Nov** | Evaluation: tests, edge cases, quality, security, and measurable outcomes |
| **21 Nov** | Hands-on: delegate or simulate a small change; inspect diff and test results |
| **22 Nov** | Mixed practice questions; log every uncertain answer as a topic to revisit |
| **23 Nov** | Review weak areas; make a one-page map of the agent workflow |
| **24 Nov** | Hands-on MCP/customization review; verify current product details in official docs |
| **25 Nov** | Timed practice; review incorrect answers and why the distractors are wrong |
| **26 Nov** | Light recall only: quick-recall section, exam logistics, rest |
| **27 Nov** | Exam day: scan carefully for scope, permissions, verification, and human-review clues |

## 11. Official references

- [GitHub Learn — Certifications](https://learn.github.com/certifications)
- [GitHub Docs — Copilot cloud agent](https://docs.github.com/en/copilot/how-tos/use-copilot-agents/cloud-agent)
- [GitHub Docs — Model Context Protocol (MCP)](https://docs.github.com/en/copilot/concepts/customizations/mcp)
- [GitHub Docs — Extending Copilot Chat with MCP servers](https://docs.github.com/en/copilot/how-tos/copilot-in-your-ide/customize-copilot/extend-copilot-with-tools-and-context/extend-copilot-chat-with-mcp)

**Last prepared:** 11 November 2026. Product capabilities and exam objectives can change; confirm details in GitHub Learn shortly before the exam.
