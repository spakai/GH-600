# GH-600 Master Cheat Sheet

[Core cheat sheet](#1-agentic-ai-the-core-loop) · [Practice gaps by test](#practice-gaps-by-test) · [Practice X.1](#practice-x1--22-missed-questions-from-another-exam) · [Practice X.2](#practice-x2--15-missed-questions)

The core reference comes from the [GitHub cheat sheet](https://github.com/spakai/GH-600/blob/main/cheat-sheet.md). The practice review at the end contains 96 entries across eight sets, including repeated topics.

**Exam date: 27 November 2026**
**Purpose:** fast revision of agentic development concepts and GitHub Copilot workflows.

Practice with the [interactive quiz](https://github.com/spakai/GH-600/blob/main/quiz.html): 45 original questions with explanations for every answer. Progress lasts for the current page session.

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
- **MCP server vs. tool:** the server is the provider/connection; its tools are individual callable operations. One server can expose multiple tools (and resources or prompts); configuring the server does not itself grant the agent permission to call every tool.
- MCP extends the agent's available tools/data; it does not replace task instructions or grant safe judgment.
- Before enabling a server, verify its source, requested permissions, configuration, and data handling. Use least privilege and avoid exposing secrets.
- In custom-agent frontmatter, the agent-level `tools` list is the allowlist. An empty `tools: []` prevents tool use; a server-level `tools: ["*"]` does not override that list. Add only the required MCP tool names to the agent's list to grant narrowly scoped access.
- **Example:** a `release-notes` server offers `lookup_release` and `publish_release`. An issue-triage agent that only needs release metadata should allow `release-notes/lookup_release`, not `release-notes/publish_release`:

  ```yaml
  ---
  name: release-issue-router
  description: Triage release issues and prepare metadata notes
  target: github-copilot
  tools:
    - read
    - search
    - release-notes/lookup_release
  mcp-servers:
    release-notes:
      type: local
      command: npx
      args: ["@example/release-notes-mcp"]
      tools: ["*"]
      env:
        RELEASE_NOTES_TOKEN: ${{ secrets.COPILOT_MCP_RELEASE_NOTES_TOKEN }}
  ---
  Triage issues, inspect related source, and use release metadata only when needed.
  ```

  The namespaced tool name is illustrative; use the actual name exposed by the server. Agent-profile `mcp-servers` configuration is for GitHub Copilot cloud agents, not custom agents running directly in IDEs.

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

**Draft PR:** explicitly marked as work in progress. GitHub shows a **Draft** label and a **Ready for review** button; it cannot be merged until marked ready. Regular PRs are ready for review when opened, but still need to satisfy required checks, approvals, and branch rules. Draft status is a readiness signal, not a safety guarantee.

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

**Concurrency-style control:** `concurrency` limits how many tasks can execute at once; `queue:max` limits how many additional tasks can wait. For example, `concurrency: 4` and `queue:max: 100` allow 4 active tasks and up to 100 queued tasks. Requests beyond the queue limit require an overflow policy such as rejection, dropping, or backpressure.

**Rehydrate state:** restore persisted workflow context—such as conversation history, task progress, tool results, artifacts, and pending decisions—so an agent can continue after a pause, restart, failure, or handoff. Rehydrating reconstructs execution state; it is more than replaying the original prompt.

**Opaque internal state:** provider-managed state whose structure is hidden from the caller. Preserve and return it as an opaque token, but do not parse, edit, or depend on its format. If persisted state must be understood, validated, or migrated by application code, use an explicit versioned schema instead.

### Copilot CLI permission reminder

A narrow prompt does not narrow the credentials or tools available to the process. Use scoped credentials, grant only the needed tools, and verify CLI flags against the installed version.

```bash
copilot --allow-tool='shell(git:*)' --deny-tool='shell(git push)'
```

Allow/deny controls illustrate tool restrictions; they are not a complete sandbox or a substitute for credential scoping.

For an unattended prompt run in CI, add `--no-ask-user` so the agent does not pause to request more user input:

```bash
copilot -p "Run the requested checks" --no-ask-user
```

This prevents follow-up questions; it does not grant tool permissions or replace explicit permission configuration.

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

**`GITHUB_OUTPUT` vs. `GITHUB_ENV`:**

| File | Purpose | How a later step reads it |
|---|---|---|
| `GITHUB_OUTPUT` | Set a named output on the current step | `steps.<step-id>.outputs.<name>` |
| `GITHUB_ENV` | Add an environment variable for subsequent steps in the same job | `$NAME` or `${{ env.NAME }}` |

Writes to either file do not change the current step. Use outputs for explicit step dependencies and environment files for ambient job configuration. To share data across jobs, expose a job output or use another supported mechanism; `GITHUB_ENV` variables do not cross job boundaries automatically.

```yaml
- id: build
  run: echo "artifact=app.zip" >> "$GITHUB_OUTPUT"
- run: echo "${{ steps.build.outputs.artifact }}"

- run: echo "BUILD_DIR=dist" >> "$GITHUB_ENV"
- run: echo "$BUILD_DIR"
```

To gate production deployment by humans, use a protected **GitHub Actions environment** with required reviewers:

```yaml
jobs:
  deploy-production:
    environment:
      name: production
    steps:
      # deployment steps
```

Configure the `production` environment's protection rules to require designated reviewers. The deployment job pauses until an authorized reviewer approves it, providing an enforced and auditable gate. A workflow `if:` condition or comment is not a substitute for an environment approval gate.

| Syntax / clue | Meaning |
|---|---|
| `uses:` | Invoke an action |
| `run:` | Execute a shell command |
| `with:` | Supply action inputs |
| `opened` / `synchronize` | PR created / PR head updated |
| `ready_for_review` | Draft PR marked ready |
| `workflow_dispatch` | Explicit workflow dispatch |
| `repository_dispatch` | External API-triggered custom event |
| `workflow_run` | React to another workflow run |
| `workflow_call` | Reusable workflow called by another workflow |

**Git hooks vs. GitHub Actions triggers:** Git hooks are local scripts run by Git during operations such as `commit` or `push` (for example, `pre-commit`); they are usually stored in `.git/hooks/` and are not shared by committing the repository. GitHub Actions triggers are workflow `on:` events such as `push` or `pull_request`; they run jobs on a GitHub-hosted or configured self-hosted runner and are defined in versioned workflow files. Local hooks help with immediate developer feedback, while Actions provide shared CI; do not rely on client-side hooks alone to enforce policy.

**`workflow_*` use cases:**

| Trigger | Example use cases | Key distinction |
|---|---|---|
| `workflow_dispatch` | Manually test Copilot setup steps; start an on-demand deployment with chosen inputs | Started by a user or API; the workflow file must exist on the default branch. Copilot runs its setup separately when an agent session starts. |
| `repository_dispatch` | Start a workflow from an external system with custom event data | Read caller-supplied JSON fields from `github.event.client_payload.<field>`; the request also supplies an `event_type` matched by `types:`. Validate payload values before using them in privileged actions. |
| `workflow_call` | Share a standard build-and-test workflow across repositories; reuse a common security scan in multiple pipelines | The caller invokes the reusable workflow at the job level with `uses:`; inputs and secrets can be passed explicitly. |
| `workflow_run` | Create an issue after validation fails; deploy after a build succeeds | Reacts to another workflow's run; `completed` includes failures, so check the upstream conclusion before a success-only or failure-only action. |

**Matrix strategy:** `strategy.matrix` creates a separate job for each combination of declared values, such as each operating system and runtime version. Use `include` and `exclude` to add or remove specific combinations.

**Matrix fan-out limit:** GitHub Actions allows at most 256 matrix-generated jobs per workflow run. A matrix with 300 agent entries exceeds that limit; batch the work across workflow runs and account for runner capacity and cost. `max-parallel` limits simultaneous jobs but does not raise the total limit.

**Concurrency with `cancel-in-progress: true`:** runs sharing the same `concurrency.group` are limited to one running and one pending run. A newly queued run replaces an older pending run; with `cancel-in-progress: true`, it also cancels the currently running run in that group. This helps avoid spending CI time on stale commits, but is unsuitable for work that should not be interrupted, such as deployments or other non-idempotent operations.

Include `github.workflow` in a concurrency group to namespace it per workflow: groups are repository-wide, so omitting it can make different workflows on the same PR branch cancel or replace each other's runs. `github.head_ref || github.run_id` groups PR runs by source branch and gives non-PR events a unique fallback.

**`github.event.client_payload`** is for custom data sent with a `repository_dispatch` event, for example `${{ github.event.client_payload.environment }}`. It is not the input mechanism for `workflow_dispatch`; use the `inputs` context for manually supplied workflow inputs.

Prefer `pull_request` when testing PR code. Do not execute untrusted PR code in a privileged `pull_request_target` context with secrets or write access. Read permissions limit token authority, but running PR code still executes untrusted code. Agent approval gates and repository Actions policies may also affect when CI runs.

**`github.ref`:** on a push to `main`, it is `refs/heads/main` (branch refs use `refs/heads/<branch>`). On `pull_request`, it is commonly `refs/pull/<number>/merge`, the PR's synthetic merge ref—not the source branch name. Use `github.head_ref` for the PR source branch and `github.base_ref` for the target branch. `pull_request_target` has different ref behavior: it uses the base branch ref.

To retain failure evidence, an upload step can use `if: always()`; this allows the step to run after earlier failures but does not create missing evidence. Sanitize files before uploading and set a suitable retention period.

For a required review artifact, set `if-no-files-found: error` on `actions/upload-artifact`; the upload step fails if no files match, rather than only warning.

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

### Copilot CLI and cloud-agent hooks

| Hook | When it runs | Typical use |
|---|---|---|
| `sessionStart` | Session starts or resumes | Initialization or policy guidance |
| `userPromptSubmitted` | User submits a prompt | Audit logging |
| `preToolUse` | Before a tool executes | Validate arguments or deny prohibited actions |
| `postToolUse` | After a tool executes | Record results; cannot prevent the completed action |
| `sessionEnd` | Session ends | Cleanup or session audit summary |

**Recall:** start → guide; prompt → log; pre → prevent; post → record. Store repository-level hook configuration as JSON files in `.github/hooks/*.json` at the repository root. Copilot CLI and cloud agent load hooks from this location; cloud agent supports only a subset of events, so verify event and command compatibility. Ensure scripts exist and paths resolve from the configured working directory.

**Do not confuse Copilot hooks with Git hooks:** `.github/hooks/*.json` contains Copilot repository-hook configuration. Standard Git hooks are local executable scripts stored in `.git/hooks/` (or the directory set by Git's `core.hooksPath`), not JSON configuration files. The `.github/hooks/` location is therefore product-specific, whereas Git's built-in hook mechanism is repository-local and script-based.

**Why a PowerShell-only repository hook may not protect a Copilot cloud agent:** a raw command is not a registered hook; the `.github/hooks/*.json` configuration must declare a supported event and handler. The cloud agent runs in Linux and honors the `bash` field (or the `command` fallback), not the `powershell` field. Git hooks under `.git/hooks/` are local and are not automatically carried into cloud-agent runs. Validate the hook configuration and runtime in the agent environment, and do not rely on an unverified hook as an enforcement boundary.

For cloud-agent tool control, use a repository-level `preToolUse` hook to inspect shell-tool arguments and deny prohibited commands before execution. `permissionRequest` is ineffective in the cloud agent because its tool calls are pre-approved. To block a tool, write one JSON object to stdout:

```json
{
  "permissionDecision": "deny",
  "permissionDecisionReason": "Blocked by policy: this operation is not allowed."
}
```

The reason is required with `deny`. In the cloud agent, `ask` is treated as `deny` because no user is available to answer. Exiting successfully without a decision leaves normal permission checks in effect; it does not grant unrestricted access. An escalation message does not create an issue unless a separate authorized action does so. Regex command filters are not a complete security boundary; apply scoped credentials and execution controls too. Log prompts only with appropriate access, retention, and secret redaction.

In an audit-style record, `operation_type` identifies **what kind of action occurred**—for example, `read`, `write`, `execute`, `delete`, `login`, or `permission_request`. It supports filtering, policy checks, reporting, and correlation without replacing details such as the actor, target resource, timestamp, or outcome. Keep values consistent and bounded to a documented taxonomy so audit consumers can classify events reliably.

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

## Practice gaps by test

### Practice 1 · Three questions discussed individually

The shared chat lists 15 missed questions and discusses Q35, Q42, and Q50 individually. This sheet focuses on those three. The chat does not identify which three were missed specifically in Practice 1.

#### Q35 · Run a step when the PR has a `preview` label

```yaml
if: ${{ contains(github.event.pull_request.labels.*.name, 'preview') }}
```

- **Remember** · PR → labels → every name → contains `preview`.
- **Why it works** · `labels` is a list of objects. `*.name` pulls out each label’s name. `contains()` checks for an element in that list.
- **Exam trap** · `pull_request.label` treats the collection as one field. `github.ref` is a Git ref. Both miss the label list.
- **Trigger detail** · If adding the label itself should start a new run, include the `labeled` activity type under `pull_request.types`. The step condition is evaluated only after a workflow run starts.

[GitHub Docs · Expressions](https://docs.github.com/en/actions/reference/workflows-and-actions/expressions)

#### Q42 · Caller and deployment environment define the same secret

**Answer** · The deployment job uses the environment’s `deployment_token`.

- **Remember** · Same name? The called job’s environment wins.
- **Why it works** · The reusable workflow’s deployment job sets `environment: production`. If that environment defines `deployment_token`, that job's `secrets.deployment_token` resolves to the environment value.
- **Exam trap** · Passing a secret explicitly from the caller can look like an override. The same-named environment secret still takes precedence for the job that references that environment.

[GitHub Docs · Reuse workflows](https://docs.github.com/en/actions/how-tos/reuse-automations/reuse-workflows)

#### Q50 · Reusable workflow or composite action?

| Cue | Reusable workflow | Composite action |
| --- | --- | --- |
| What it packages | One or more jobs | Multiple steps |
| Definition | Workflow YAML with `on: workflow_call` | `action.yml` with `runs.using: composite` |
| Call site | `jobs.<job_id>.uses` | `steps[*].uses` |
| Runner | Each called job selects its runner | Uses the enclosing job’s runner |

**Remember** · Workflow goes in a job slot. Composite goes in a step slot. Both use `uses`, so check its indentation.

[GitHub Docs · Reusing workflow configurations](https://docs.github.com/en/actions/concepts/workflows-and-actions/reusing-workflow-configurations)

#### Quick recall

1. Which expression extracts every PR label name before checking membership?
2. A caller passes `deployment_token` and the called job’s environment has the same secret. Which value does the job use?
3. You want build and deployment to run as two jobs on different runners. Which reuse mechanism fits, and where does `uses` go?

**Answers** · 1. `contains(github.event.pull_request.labels.*.name, 'preview')` for the `preview` label. 2. The environment’s value. 3. A reusable workflow, invoked through `jobs.<job_id>.uses`.

[Shared study chat](https://chatgpt.com/share/6b07d09f-b8e4-83ec-bffb-191c05991557)

### Practice 2 · 14 missed questions

#### Copilot hooks and approval gates

| Question | Answer and memory cue |
| --- | --- |
| **Q14 · Block a tool call** | Write a JSON object to stdout from `preToolUse`. Set `permissionDecision` to `deny` and include `permissionDecisionReason`. |
| **Q15 · Repository hook location** | Put repository hook configuration under `.github/hooks/*.json`. Remember **hooks → JSON**, Actions workflows → YAML. |
| **Q22 · Block a direct shell push in cloud agent** | Use `preToolUse` to evaluate the proposed command before execution. The practice test lists `permissionRequest` as a CLI-only distractor. |
| **Q57 · Copilot PR checks haven't started** | Look for **Approve and run workflows**. With the default approval policy, a reviewer with write access reviews the changes and starts the workflows. |
| **Q73 · Empty hook output** | Default behavior continues if no other hook denies the call. Empty output leaves normal permission checks in effect. |
| **Q80 · Matcher `bash\|powershell`** | The practice answer treats the matcher as a full-match regex on the event-specific value. Read it as exactly `bash` or exactly `powershell`. |

Example denial for Q14 and Q22

```json
{
  "permissionDecision": "deny",
  "permissionDecisionReason": "Open a pull request instead of pushing directly."
}
```

**Cloud agent cue** · GitHub documents that `ask` is treated as `deny` on cloud agent because there is no interactive user to answer.

[GitHub Docs · Hooks reference](https://docs.github.com/en/copilot/reference/hooks-reference) · [GitHub Docs · Cloud agent settings](https://docs.github.com/en/copilot/how-tos/use-copilot-agents/cloud-agent/configuring-agent-settings)

#### Copilot CLI and MCP

| Question | Answer and memory cue |
| --- | --- |
| **Q18 · Unattended `copilot -p`** | `--no-ask-user` disables the `ask_user` tool. Tool permissions are configured separately. |
| **Q37 · MCP server key in JSON** | The quiz expects `mcpServers` for portable Copilot MCP JSON. The editor-managed VS Code `.vscode/mcp.json` format uses `servers`. Check the configuration surface before choosing the key. |
| **Q65 · `mcp-servers`** | Hyphenated `mcp-servers` belongs to supported custom agent YAML frontmatter. |
| **Q76 · Default CLI process logs** | On macOS and Linux, look under `~/.copilot/logs/`. Think **CLI process → user home**. |
| **Q83 · Agent cannot read an external tracker** | Configure the tool or MCP integration and grant the access needed to read the tracker. A prompt can state the task, while the integration provides access. |
| **Q94 · MCP server entry has `url` and no `command`** | Remote HTTP or SSE, depending on the endpoint and client configuration. A local stdio entry launches a process with `command`. |

[GitHub Docs · CLI flags](https://docs.github.com/en/copilot/reference/copilot-cli-reference/cli-command-reference) · [GitHub Docs · CLI config directories](https://docs.github.com/en/copilot/reference/copilot-cli-reference/cli-config-dir-reference) · [GitHub Docs · Custom agents](https://docs.github.com/en/copilot/reference/custom-agents-configuration) · [VS Code · MCP configuration](https://code.visualstudio.com/docs/agents/reference/mcp-configuration)

#### PR validation

| Question | Answer and memory cue |
| --- | --- |
| **Q41 · Checkout `github.head_ref`** | It names the PR **source branch**. `github.base_ref` names the **target branch**. |
| **Q60 · Keep the latest validation for the same PR branch** | Use workflow-level `concurrency` with a stable group such as `${{ github.workflow }}-${{ github.head_ref }}` and `cancel-in-progress: true`. `queue: max` cannot be combined with `cancel-in-progress: true`. For a group scoped to an individual PR, see Practice 3 Q58. |

[GitHub Docs · Actions contexts](https://docs.github.com/en/actions/reference/workflows-and-actions/contexts) · [GitHub Docs · Concurrency](https://docs.github.com/en/actions/how-tos/write-workflows/choose-when-workflows-run/control-workflow-concurrency)

### Practice 3 · Two missed questions

#### Q56 · Where an uploaded artifact lives

**Answer** · GitHub Actions artifact storage associated with the workflow run.

- **Remember** · A runner creates the file. `actions/upload-artifact` uploads it to the workflow run.
- **Exam trap** · Uploading a report or build output does not create a Git commit. The uploaded copy is available after the job ends, subject to retention.
- **Where to find it** · Open the workflow run’s summary and look under **Artifacts**.

[GitHub Docs · Workflow artifacts](https://docs.github.com/en/actions/concepts/workflows-and-actions/workflow-artifacts)

#### Q58 · Cancel older validation runs for the same PR

**Answer** · Group by PR number and enable cancellation. Use this at the workflow level for a PR-only validation workflow.

```yaml
on:
  pull_request:

concurrency:
  group: pr-validation-${{ github.event.pull_request.number }}
  cancel-in-progress: true
```

| Candidate | What it groups |
| --- | --- |
| `github.event.pull_request.number` | All pushes to the same PR. Different PRs get different groups. |
| `github.run_id` | Each workflow run has its own ID. New runs get separate groups. |
| `github.run_attempt` | The attempt number starts at 1 for each run. Unrelated first attempts can collide. |
| `runner.os` | The runner’s OS identifies a platform and is unavailable at workflow-level concurrency. |

**Remember** · Group by what should replace earlier work. Here that’s the **PR**. Q60 gives the branch-based exam answer. Q58 uses the PR number to avoid grouping unrelated PRs with the same source branch name.

[GitHub Docs · Concurrency](https://docs.github.com/en/actions/how-tos/write-workflows/choose-when-workflows-run/control-workflow-concurrency)

#### Quick recall · Practices 2 and 3

1. Which hook and JSON field block a tool call before it executes?
2. What happens when the hook prints nothing?
3. What are the MCP server keys for portable JSON, editor-managed VS Code JSON, and custom agent YAML?
4. Where does an uploaded report live, and what identifies the concurrency group that should cancel stale checks for one PR?

**Answers** · 1. `preToolUse` and `permissionDecision: deny`, with a reason. 2. Normal permission flow continues. 3. `mcpServers`, `servers`, and `mcp-servers`, respectively. 4. In the workflow run’s artifact storage. Use the PR number as the group identifier.

### Practice 4 · Nine missed questions

#### Copilot CLI

| Question | Answer and memory cue |
| --- | --- |
| **Q4 · Export a completed programmatic session to Markdown** | `--share=PATH` writes the transcript locally. Without a custom path, the default is `./copilot-session-<ID>.md`. `--resume` selects a session and `--log-level` changes diagnostic verbosity. |
| **Q6 · Interactive authentication** | **Quiz answer** · OAuth device flow. **Current GitHub docs** · A local terminal recommends browser sign-in. A remote or headless terminal uses device-code sign-in. Start with `copilot login` or `/login`. |
| **Q17 · Ignore custom instruction files for one invocation** | `--no-custom-instructions` skips instruction sources such as `AGENTS.md`. Built-in MCP flags control servers. |
| **Q40 · No questions vs. Autopilot** | `--no-ask-user` disables the questioning tool. Autopilot can continue across model turns. Tool permissions are a separate control. See Practice 2 Q18. |

[GitHub Docs · CLI flags](https://docs.github.com/en/copilot/reference/copilot-cli-reference/cli-command-reference) · [GitHub Docs · CLI authentication](https://docs.github.com/en/copilot/how-tos/copilot-cli/set-up-copilot-cli/authenticate-copilot-cli) · [GitHub Docs · Autopilot](https://docs.github.com/en/copilot/how-tos/copilot-cli/use-copilot-cli/use-autopilot-mode)

#### Actions security and validation

| Question | Answer and memory cue |
| --- | --- |
| **Q11 · Binary build provenance** | Set `contents: read`, `id-token: write`, and `attestations: write` under `permissions`. Read source, request an OIDC token, create the attestation. |
| **Q14 · What `id-token: write` grants** | The job can request a GitHub OIDC token. The cloud provider's trust policy controls token exchange, and the resulting role controls cloud access. |
| **Q25 · Required CI check in a merge queue** | Include `merge_group` in the CI triggers alongside `pull_request` as appropriate. The queue needs the check on its temporary merge-group commit. |
| **Q26 · Stop a hung test after 20 minutes** | Set `timeout-minutes: 20` on the test step. Keep the default failure behavior by leaving `continue-on-error` unset. |
| **Q47 · Pass an untrusted PR title to a shell** | Assign the PR title to a step-level `env` variable. Read that variable in quotes and use a fixed `printf` format. Direct expression interpolation in `run` inserts the value into the script source. |

```yaml
- name: Print PR title
  env:
    PR_TITLE: ${{ github.event.pull_request.title }}
  run: printf '%s\n' "$PR_TITLE"
```

[GitHub Docs · Attestations](https://docs.github.com/en/actions/how-tos/secure-your-work/use-artifact-attestations/use-artifact-attestations) · [GitHub Docs · OIDC](https://docs.github.com/en/actions/reference/security/oidc) · [GitHub Docs · Merge queue](https://docs.github.com/en/repositories/configuring-branches-and-merges-in-your-repository/configuring-pull-request-merges/managing-a-merge-queue) · [GitHub Docs · Workflow syntax](https://docs.github.com/en/actions/reference/workflows-and-actions/workflow-syntax) · [GitHub Docs · Secure use](https://docs.github.com/en/actions/reference/security/secure-use)

### Practice 5 · 15 missed questions

| Question | Answer and memory cue |
| --- | --- |
| **Q2 · Did the security job fail?** | `${{ needs.security.result == 'failure' }}` checks the named dependency. To let the notification job itself run after a dependency failure, use `${{ failure() && needs.security.result == 'failure' }}`. |
| **Q10 · Autopilot and session selection** | **Quiz answer** · `copilot --autopilot --resume abc123` is the invalid choice among the original options. Learn the tested pair and check the command reference for the installed CLI when applying it. |
| **Q22 · Protected production deployment** | Set `environment: production` on the deployment job. The environment supplies its configured protection rules and secrets. |
| **Q35 · PR has a preview label** | `${{ contains(github.event.pull_request.labels.*.name, 'preview') }}`. See Practice 1 Q35. |
| **Q36 · 20 matrix jobs, four at once** | Set `strategy.max-parallel: 4`. This caps simultaneous jobs. |
| **Q40 · Cloud agent MCP capabilities** | Tools only on the tested cloud agent surface. A tool is a callable operation, a resource provides data, and a prompt is a template. |
| **Q42 · Same secret name in caller and environment** | The called deployment job uses the environment’s value. See Practice 1 Q42. |
| **Q44 · Manual selection and read-only capabilities** | The quiz uses `infer: false` and a narrow read/search tool list. Current profiles can express manual-only selection with `disable-model-invocation: true` and `user-invocable: true`. |
| **Q45 · Invalid key with job-level `uses`** | `runs-on`. The called workflow defines its own jobs and runners. |
| **Q47 · Matrix from discovery JSON** | Set `strategy.matrix` to `${{ fromJSON(needs.discover.outputs.matrix) }}` and declare `needs: discover`. Parse the entire object containing `include`. |
| **Q48 · Validate PRs targeting main before merge** | Use `pull_request` with `branches: [main]`. The filter matches the PR’s target branch. |
| **Q49 · `LOG_LEVEL` at three scopes** | The step-level value wins inside that step. Precedence is step → job → workflow. |
| **Q50 · Reusable workflow vs. composite action** | Reusable workflow → jobs → job-level `uses`. Composite action → steps → step-level `uses`. See Practice 1 Q50. |
| **Q53 · Allow only the experimental runtime to fail** | Set job-level `continue-on-error: ${{ matrix.experimental }}` with `experimental: true` only for the experimental entry. `fail-fast` controls cancellation of other matrix jobs after a mandatory failure. |
| **Q54 · Write an `image_tag` step output** | Append `image_tag=value` to `$GITHUB_OUTPUT`. With step ID `meta`, read it as `steps.meta.outputs.image_tag`. |

[GitHub Docs · Expressions](https://docs.github.com/en/actions/reference/workflows-and-actions/expressions) · [GitHub Docs · Workflow syntax](https://docs.github.com/en/actions/reference/workflows-and-actions/workflow-syntax) · [GitHub Docs · Custom agents](https://docs.github.com/en/copilot/reference/custom-agents-configuration) · [GitHub Docs · CLI flags](https://docs.github.com/en/copilot/reference/copilot-cli-reference/cli-command-reference)

### Practice 6 · 16 missed questions

#### Triggers, containers, and deployments

| Question | Answer and memory cue |
| --- | --- |
| **Q1 · Release only on tags** | `${{ startsWith(github.ref, 'refs/tags/') }}`. `github.ref` holds the ref path. `github.sha` holds a commit ID. |
| **Q5 · Chain with `workflow_run`** | Up to three downstream levels after the first workflow. `A → B → C → D` can run. This chain will not trigger `E`. |
| **Q19 · Run job steps in Docker** | Use `jobs.<id>.container.image: node:22` as the nested configuration path. `services` configures sidecars. |
| **Q20 · Tell a PR comment from an issue comment** | In an `issue_comment` workflow, check `${{ github.event.issue.pull_request }}`. That property identifies a PR conversation comment. |
| **Q22 · Composite action `run` step** | Set an explicit `shell` on each `run` step. The caller job supplies the runner. |
| **Q34 · Prevent administrator bypass in production** | Disable **Allow administrators to bypass configured protection rules** in the environment’s settings. |
| **Q38 · Preserve a manual input as Boolean** | Use `${{ inputs.perform_deploy }}`. `github.event.inputs` exposes values as strings. |
| **Q39 · Default shell inside a job container** | `sh`. Specify `shell: bash` when you need Bash features and the image provides Bash. |
| **Q46 · Staging secrets and variables without deployment record** | Use environment object syntax with `name: staging` and `deployment: false`. |

```yaml
environment:
  name: staging
  deployment: false
```

[GitHub Docs · Workflow events](https://docs.github.com/en/actions/reference/workflows-and-actions/events-that-trigger-workflows) · [GitHub Docs · Workflow syntax](https://docs.github.com/en/actions/reference/workflows-and-actions/workflow-syntax) · [GitHub Docs · Action metadata](https://docs.github.com/en/actions/reference/workflows-and-actions/metadata-syntax) · [GitHub Docs · Deployment environments](https://docs.github.com/en/actions/how-tos/write-workflows/choose-what-workflows-do/deploy-to-environment)

#### Workflow commands

| Question | Answer and memory cue |
| --- | --- |
| **Q21 · File and line error annotation** | `echo "::error file=src/app.js,line=12::Invalid configuration"`. The properties identify the location and the text after the second `::` is the message. |
| **Q30 · Collapsible log section** | Print `::group::A heading` before the lines and `::endgroup::` after them. |
| **Q42 · Print untrusted text that contains command markers** | Print `::stop-commands::<random-token>`, print the text, then print `::<random-token>::` to re-enable processing. Use a fresh, unpredictable token for each run. Practice 4 Q47 protects the shell input path. This protects the workflow-command output path. |
| **Q47 · Mask a runtime-generated secret** | Print `::add-mask::<value>` before the value appears in logs. Masking applies to subsequent output. |

[GitHub Docs · Workflow commands](https://docs.github.com/en/actions/reference/workflows-and-actions/workflow-commands)

#### Copilot agents, inventory, and MCP secrets

| Question | Answer and memory cue |
| --- | --- |
| **Q13 · Personal and repo agent share a filename** | The repository agent under `.github/agents/` takes precedence over the same-named personal agent in Copilot CLI. |
| **Q40 · List resources discovered for the working directory** | `copilot plugins list` is the quiz answer for the inventory. Current CLI docs list dedicated commands, including `copilot instruction list` for instruction sources and `copilot lsp list` for language servers. Check the command reference for your installed version. |
| **Q44 · Cloud agent secret substitution in repository MCP** | Use a name such as `COPILOT_MCP_VENDOR_API_KEY`. The `COPILOT_MCP_` prefix identifies an eligible Agents secret or variable. |

[GitHub Docs · Copilot CLI commands](https://docs.github.com/en/copilot/reference/copilot-cli-reference/cli-command-reference) · [GitHub Docs · Custom agents](https://docs.github.com/en/copilot/how-tos/copilot-cli/use-copilot-cli/invoke-custom-agents) · [GitHub Docs · Repository MCP servers](https://docs.github.com/en/enterprise-cloud@latest/copilot/how-tos/copilot-on-github/customize-copilot/configure-mcp-servers)

### Practice X.1 · 22 missed questions from another exam

#### Plan approval, scope, and handoffs

| Question | Expected answer and memory cue |
| --- | --- |
| **Q6 · What does a PR plan check enforce?** | **Yes / No / Yes.** A PR workflow can check the plan file and the headings it tests. Instructions guide Copilot, and a required status check can block merge. Neither proves the plan was written before the first edit. |
| **Q7 · Review a plan before implementation** | **3 → 5 → 4 → 2.** Define acceptance criteria, constraints, and allowed files → request a plan-only response → review scope, risks, and tests → record approval or requested changes in the issue. |
| **Q16 · Why did the reviewer fail the check?** | The implementer contradicted the planner’s handoff. The plan ruled out schema changes, but the diff added a migration. Successful jobs can still produce an outcome that violates the approved scope. |
| **Q30 · Reconstruct a multi-agent decision** | **2 → 4 → 3 → 1.** Define required decision fields → capture each agent outcome as it completes → write the final coordination summary to `$GITHUB_STEP_SUMMARY` → upload raw handoffs as an artifact. |
| **Q47 · Select only the tools required for a pilot** | **3 → 1 → 4 → 5.** Inventory repo actions, checks, and external data needs → configure the agent’s tool allowlist → install build and test dependencies in Copilot setup steps → run a pilot issue and inspect the session output. |
| **Q66 · Turn a vague checkout task into an agent-ready issue** | **4 → 6 → 1 → 3.** Identify affected paths, constraints, and allowed tools → define expected outputs → specify evidence such as checks, tests, and logs → create the GitHub issue. |
| **Q69 · Resume from approved state** | **Yes / Yes / No.** The saved state SHA matches the current PR head. Exclude previous failed-run artifacts and rejected local notes. Resume with the approved plan, current diff, failed required checks, and runbook. |

#### Release branches, workflow gates, and permissions

| Question | Expected answer and memory cue |
| --- | --- |
| **Q22 · Fix only `release/2026.05`** | Select `release/2026.05` as the base branch when assigning the work. It supplies the starting point and PR target. |
| **Q23 · Release-specific instructions and validation** | **Quiz answer · Yes / Yes / No.** A PR into `release/2026.05` matches `release/**`. The quiz says code review reads instructions from the base branch. Current GitHub docs say it reads them from the **head branch**. The workflow’s branch filter controls triggering and cannot prevent branch creation from `main`. |
| **Q29 · Manual rollback to a known-good SHA** | **Yes / No / Yes.** `workflow_dispatch` collects the SHA, checkout selects it, and the `staging` environment can require approval. With the shown permissions, `gh issue create` still lacks `issues: write`. Pass the SHA to shell commands via a quoted environment variable. |
| **Q45 · Release validation check stays pending** | The PR targets `main`, which does not match `release/**`. For `pull_request.branches`, inspect the base branch first. A required check can stay pending when its workflow is skipped by a filter. |
| **Q49 · Post a PR evaluation comment** | **Quiz fill-ins · `contents: read`, `pull-requests: write`, `issues: read`.** The first permission covers checkout and the second permits a PR timeline comment. GitHub’s comment endpoint accepts either PR-write or issue-write permission. The snippet must also supply `PR_NUMBER` for the command to target a PR. |
| **Q67 · Dependency review trigger and permissions** | **Quiz fill-ins · `.github/dependabot.yml`, `contents: read`, `pull-requests: read`.** Add the Dependabot path to the manifest and lockfile filters. If this is an always-required check, account for skipped workflows leaving the check pending on unrelated PRs. |
| **Q70 · Manual protected production release** | **Fill-ins · `workflow_dispatch`, `production`, `./scripts/deploy_checkout.sh`.** The environment controls approval and grants access to its secrets after protection rules pass. The file-presence check only confirms the approval record exists. |

[GitHub Docs · Copilot code review](https://docs.github.com/en/copilot/concepts/agent-capabilities/agent-powered-workflows-and-operations/code-review) · [Workflow syntax](https://docs.github.com/en/actions/reference/workflows-and-actions/workflow-syntax) · [Issue and PR timeline comments](https://docs.github.com/en/rest/issues/comments#create-an-issue-comment) · [Deployment environments](https://docs.github.com/en/actions/how-tos/write-workflows/choose-what-workflows-do/deploy-to-environment)

#### MCP registry discovery and enforcement

| Question | Expected answer and memory cue |
| --- | --- |
| **Q19 · Registry host, version routes, and local servers** | **Yes / Yes / No.** Configure the registry base URL. The service must expose the server list and version lookups. In the quiz scenario, registry-only enforcement covers local servers too. |
| **Q26 · Registry gallery returns 404** | The request contains the endpoint path twice. Use the registry base URL in policy so the client can append `/v0.1/servers` once. |
| **Q43 · Match registry settings to their purpose** | **1-C, 2-B, 3-A, 4-D.** Registry URL → catalog discovery. Registry only → restrict access to listed servers. `GET /v0.1/servers` → list entries. Exact local server ID match → keep that approved local server usable in the quiz scenario. |
| **Q48 · Keep approved local servers under registry-only policy** | Register each approved local server under its exact ID before enforcing the restriction. |

The three v0.1 routes to remember are `/v0.1/servers`, `/v0.1/servers/{serverName}/versions/latest`, and `/v0.1/servers/{serverName}/versions/{version}`.

[GitHub Docs · Configure an MCP registry](https://docs.github.com/en/copilot/how-tos/administer-copilot/manage-mcp-usage/configure-mcp-registry) · [Restrict access by registry](https://docs.github.com/en/copilot/how-tos/administer-copilot/manage-mcp-usage/restrict-based-on-registry)

#### Copilot Memory and CLI context

| Question | Expected answer and memory cue |
| --- | --- |
| **Q31 · Remove an outdated repository fact** | Delete the misleading fact from repository Copilot Memory. The stale migration guidance is stored at repository scope. |
| **Q32 · Start CLI from the right directory** | Launch Copilot CLI from the trusted repository root so project context starts in the target repo. Avoid starting from the shared parent with unrelated exports, logs, and old copies. |
| **Q33 · Reset a session stuck on a rejected approach** | Reset the relevant local CLI session context, for example by starting a fresh session without resuming the old one. Repository-level Memory is independent. |
| **Q56 · Match each memory control to its rule** | **1-B, 2-A, 3-C, 4-D.** Unused entry → quiz says expiry after **28 days**. Misleading repo fact → delete manually. Stale session context → reset locally. Custom instructions → persistent repo files. Current GitHub docs give **90 days by default** without reinforcement, with configurable retention for repository facts. |

[GitHub Docs · Copilot Memory](https://docs.github.com/en/copilot/concepts/prompting-and-context/copilot-memory) · [Copilot CLI command reference](https://docs.github.com/en/copilot/reference/copilot-cli-reference/cli-command-reference)

### Practice X.2 · 15 missed questions

#### Approval gates and review

| Question | Expected answer and memory cue |
| --- | --- |
| **Q3 · Match escalation signals to responses** | **1-A, 2-B, 3-C, 4-D.** Failed required check → review before marking ready. Denied shell command → use the approval path with the blocked command and its context. Missing MCP tools → inspect server startup and setup. Out-of-scope edits → narrow the issue and request review. |
| **Q8 · Manual run started but the agent did not** | The approval gate blocked the agent-start job because the linked issue lacked the required `agent-approved` label. Inspect the gate result before investigating the agent itself. |
| **Q9 · Protected agent-execution environment** | **Yes / No / Yes.** Required environment reviewers can hold the job before protected steps run and provide a human execution gate. A manual trigger records that someone started the workflow. Plan approval requires its own evidence. |
| **Q29 · Migration PR fails a credential-sensitive test** | Keep the PR in draft and request owner review. This preserves the work and gives the responsible owner a chance to evaluate the test failure and credential request before advancing the PR. |
| **Q38 · Payments refactor also changes workflow YAML** | Review the draft diff and require owner approval before merging. Passing unit tests and an approved issue cover different decisions from approving a workflow change. |
| **Q55 · Find the first broken handoff** | Implementation touched the payment gateway outside the checkout retry scope. The payment test failure came next, and release review skipped because validation failed. Read the approved scope, changed paths, and job outcomes in that order. |

#### Agent profiles, MCP access, and CLI scope

| Question | Expected answer and memory cue |
| --- | --- |
| **Q10 · Cloud custom agent with manual selection** | **Fill-ins · `github-copilot`, `tools`, `disable-model-invocation`.** Set `target: github-copilot`, list `["read", "search", "edit", "github/*"]` under `tools`, and set `disable-model-invocation: true`. Keep user invocation enabled. The pasted quiz duplicates choices B and C with the same values. |
| **Q12 · Match tool allowlist patterns** | **1-A, 2-B, 3-C, 4-D.** Omitted `tools` → all available configured tools. `tools: []` → zero tools. Explicit aliases → listed tools only. `github/*` → the GitHub MCP namespace. |
| **Q17 · Dirty worktree before local agent changes** | Cleanly isolate existing unrelated edits before execution, for example with a separate worktree. Check the diff before staging so the PR contains only the intended task. |
| **Q37 · MCP server configured with an empty tool list** | **Yes / No / Yes.** An empty list disables tools. Declaring the server does not grant its tools to the agent. Add the specific server tools that the task needs to the allowlist. |
| **Q40 · Enterprise registry-only policy** | **Yes / Yes / No.** On supported surfaces, registry-only restricts use to approved entries. The quiz applies exact ID matching to local servers. The quiz’s third statement is false because ID matching offers limited protection against edited local configuration. See Practice X.1 Q19, Q43, and Q48. |
| **Q48 · CLI launched from the home directory** | The CLI started with an overly broad working scope. Launch from the trusted target repository root so unrelated home-directory data stays outside the intended task. |

[GitHub Docs · Custom agent configuration](https://docs.github.com/en/copilot/reference/custom-agents-configuration) · [Registry restrictions](https://docs.github.com/en/copilot/how-tos/administer-copilot/manage-mcp-usage/restrict-based-on-registry)

#### Durable evidence and workflow comments

| Question | Expected answer and memory cue |
| --- | --- |
| **Q43 · Preserve diagnostics when tests fail** | **Fill-ins · `pull_request`, `always()`, `actions/upload-artifact@v4`.** Run validation for PRs into `main` and upload whatever diagnostics exist even after a failed step. In the quiz snippet, the plan copy follows the test command inside the same script. Move the copy before the test, or into its own failure-tolerant step, so a failed test cannot skip it. |
| **Q54 · Prevent context drift across issue, PR, and CI** | **2 → 3 → 4 → 6.** Put scope and acceptance criteria in the issue → record decisions in the draft PR → upload validation evidence → review the durable records before marking ready. |
| **Q62 · Comment only on guardrail failure** | **Fill-ins · `pull_request`, `pull-requests`, `run`.** Use `contents: read` and `pull-requests: write` with `gh pr comment`. The quiz uses `if: failure()`, which also catches other earlier step failures. For a comment tied to the guardrail specifically, give that step `id: guardrail` and use `if: failure() && steps.guardrail.outcome == 'failure'`. If the script can fail for reasons other than a violation, have it report a separate violation signal. |

[GitHub Docs · Expressions and status checks](https://docs.github.com/en/actions/reference/workflows-and-actions/expressions) · [Workflow artifacts](https://docs.github.com/en/actions/concepts/workflows-and-actions/workflow-artifacts)
