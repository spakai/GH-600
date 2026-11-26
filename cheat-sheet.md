# GH-600 Master Cheat Sheet

[Core cheat sheet](#1-agentic-ai-the-core-loop) · [Topic-based review](#topic-based-review)

The core reference comes from the [GitHub cheat sheet](https://github.com/spakai/GH-600/blob/main/cheat-sheet.md). The topic-based review at the end covers Copilot CLI, MCP, agent guardrails, and GitHub Actions.

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

## 10. Official references

- [GitHub Learn — Certifications](https://learn.github.com/certifications)
- [GitHub Docs — Copilot cloud agent](https://docs.github.com/en/copilot/how-tos/use-copilot-agents/cloud-agent)
- [GitHub Docs — Model Context Protocol (MCP)](https://docs.github.com/en/copilot/concepts/customizations/mcp)
- [GitHub Docs — Extending Copilot Chat with MCP servers](https://docs.github.com/en/copilot/how-tos/copilot-in-your-ide/customize-copilot/extend-copilot-with-tools-and-context/extend-copilot-chat-with-mcp)

**Last prepared:** 11 November 2026. Product capabilities and exam objectives can change; confirm details in GitHub Learn shortly before the exam.

## Topic-Based Review

### Copilot CLI, sessions, and Memory

| Topic | What to remember |
| --- | --- |
| **Export a completed programmatic session to Markdown** | `--share=PATH` writes the transcript locally. Without a custom path, the default is `./copilot-session-<ID>.md`. `--resume` selects a session and `--log-level` changes diagnostic verbosity. |
| **Interactive authentication** | Start with `copilot login` or `/login`. GitHub documents browser sign-in for a local terminal and OAuth device-code sign-in for remote or headless environments. |
| **Ignore custom instruction files for one invocation** | `--no-custom-instructions` skips instruction sources such as `AGENTS.md`. Built-in MCP flags control servers. |
| **No questions vs. Autopilot** | In unattended `copilot -p` runs, `--no-ask-user` disables the questioning tool. Autopilot can continue across model turns. Tool permissions are a separate control. |
| **Default CLI process logs** | On macOS and Linux, look under `~/.copilot/logs/`. Think **CLI process → user home**. |
| **Autopilot and session selection** | Select Autopilot with `--mode autopilot`. Use `--resume` to select an existing session. For a plan followed by Autopilot, the CLI documents `--plan --mode autopilot`. Check the command reference for the CLI version you use. |
| **Personal and repo agent share a filename** | The repository agent under `.github/agents/` takes precedence over the same-named personal agent in Copilot CLI. |
| **List resources discovered for the working directory** | Use the dedicated CLI inventory commands, such as `copilot instruction list` for instruction sources and `copilot lsp list` for language servers. Older CLI versions grouped inventory under `copilot plugins list`. |
| **Remove an outdated repository fact** | Delete the misleading fact from repository Copilot Memory. The stale migration guidance is stored at repository scope. |
| **Start CLI from the right directory** | Launch Copilot CLI from the trusted repository root so project context starts in the target repo. Avoid starting from the shared parent with unrelated exports, logs, and old copies. |
| **Reset a session stuck on a rejected approach** | Reset the relevant local CLI session context, for example by starting a fresh session without resuming the old one. Repository-level Memory is independent. |
| **Match each memory control to its rule** | Copilot Memory entries expire after **90 days by default** without reinforcement. Retention for repository facts can be configured. Delete misleading repository facts manually, reset stale CLI context locally, and keep durable project guidance in repository instruction files. |
| **Dirty worktree before local agent changes** | Cleanly isolate existing unrelated edits before execution, for example with a separate worktree. Check the diff before staging so the PR contains only the intended task. |
| **CLI launched from the home directory** | The CLI started with an overly broad working scope. Launch from the trusted target repository root so unrelated home-directory data stays outside the intended task. |

### Custom agents and MCP

| Topic | What to remember |
| --- | --- |
| **Manual selection and read-only capabilities** | Set `disable-model-invocation: true` and keep `user-invocable: true` for manual selection. Restrict `tools` to the capabilities the agent needs, such as `read` and `search`. `infer: false` is the older setting. |
| **Cloud custom agent with manual selection** | **Configuration** · `github-copilot`, `tools`, `disable-model-invocation`. Set `target: github-copilot`, list `["read", "search", "edit", "github/*"]` under `tools`, and set `disable-model-invocation: true`. Keep user invocation enabled. |
| **Match tool allowlist patterns** | Omitted `tools` → all available configured tools. `tools: []` → zero tools. Explicit aliases → listed tools only. `github/*` → the GitHub MCP namespace. |
| **MCP server configured with an empty tool list** | An empty list disables tools. Declaring the server does not grant its tools to the agent. Add the specific server tools that the task needs to the allowlist. |
| **MCP server key in JSON** | Use `mcpServers` in portable Copilot MCP JSON. The editor-managed VS Code `.vscode/mcp.json` format uses `servers`. |
| **`mcp-servers`** | Hyphenated `mcp-servers` belongs to supported custom agent YAML frontmatter. |
| **Agent cannot read an external tracker** | Configure the tool or MCP integration and grant the access needed to read the tracker. A prompt can state the task, while the integration provides access. |
| **MCP server entry has `url` and no `command`** | Remote HTTP or SSE, depending on the endpoint and client configuration. A local stdio entry launches a process with `command`. |
| **Cloud agent MCP capabilities** | The cloud-agent MCP configuration exposes callable tools. In MCP terminology, resources provide data and prompts provide reusable templates. Supported capabilities depend on the Copilot surface. |
| **Cloud agent secret substitution in repository MCP** | Use a name such as `COPILOT_MCP_VENDOR_API_KEY`. The `COPILOT_MCP_` prefix identifies an eligible Agents secret or variable. |
| **Registry host, version routes, and local servers** | Configure the registry base URL. The service must expose the server list and version lookups. Registry-only enforcement covers local servers too. |
| **Registry gallery returns 404** | The request contains the endpoint path twice. Use the registry base URL in policy so the client can append `/v0.1/servers` once. |
| **Match registry settings to their purpose** | Registry URL → catalog discovery. Registry only → restrict access to listed servers. `GET /v0.1/servers` → list entries. Exact local server ID match → keep that approved local server usable. |
| **Keep approved local servers under registry-only policy** | Register each approved local server under its exact ID before enforcing the restriction. |
| **Enterprise registry-only policy** | Registry-only policy restricts supported clients to approved registry entries. Local server matching depends on the configured ID, so this check offers limited protection against edited local configuration. |

The MCP registry v0.1 routes are `/v0.1/servers`, `/v0.1/servers/{serverName}/versions/latest`, and `/v0.1/servers/{serverName}/versions/{version}`.

#### Allow one approved Sentry tool

The `mcp-servers.sentry` entry configures the connection. The agent’s `tools` list determines which tools it can call. Start read-only with `tools: ["read", "search"]`, then add the exact server tool name after approval.

```yaml
# Example if the configured Sentry server exposes get_issue
tools: ["read", "search", "sentry/get_issue"]
```

Use `sentry/*` to enable every tool exposed by that server. Shell execution stays outside both allowlists because neither includes `execute` or a shell alias.

[GitHub Docs · Custom agent tool configuration](https://docs.github.com/en/copilot/reference/custom-agents-configuration)

#### Cloud Agent and VS Code Agent

| Capability | Copilot cloud agent | VS Code IDE agent |
| --- | --- | --- |
| Execution | Hosted checkout. Delegates work asynchronously and can prepare a PR. | Active workspace and terminal, including a connected remote host or dev container. |
| MCP tools | Supported. A tool can look up an issue or return runbook text. | Supported, alongside IDE and extension tools. |
| MCP resources | Unsupported. | Supported. Attach server-provided data, such as a runbook, as context. |
| MCP prompts | Unsupported. | Supported. Invoke server-provided templates from chat. |
| Task requests and instructions | Accepts task prompts and supports repository instructions, custom agents, and skills. | Accepts task prompts and also supports reusable IDE `.prompt.md` files. |
| Environment | Uses hosted files, credentials, and network access. Local files, IDE settings, and local credentials do not automatically carry over. The firewall limits outbound access by default. | Uses the workspace environment, subject to available tools and approvals. |

**Remember** · A task prompt is your request. A `.prompt.md` file is a reusable IDE task template. An MCP prompt is a template published by a server. Cloud agent and code review support MCP tools only. For servers configured in repository settings or agent profiles, they also lack support for remote servers that use OAuth.

| Configuration scope | What it controls |
| --- | --- |
| Enterprise registry | Set the registry URL and use **Registry only** to restrict supported clients to approved servers. |
| Repository MCP JSON | Configure servers under **Settings → Copilot → MCP servers** for cloud agent and code review. |
| Custom agent profile | Select callable tools with `tools`. Use `server/tool` for one tool or `server/*` for a namespace. |
| Default GitHub MCP server | Supplies repo, issue, and PR access without custom server JSON. Its default cloud-agent token is scoped to read access in the current repository. |

[GitHub Docs · Repository MCP configuration](https://docs.github.com/en/copilot/how-tos/copilot-on-github/customize-copilot/configure-mcp-servers) · [VS Code · MCP servers](https://code.visualstudio.com/docs/agent-customization/mcp-servers)

#### Cloud Agent setup with private npm packages

Use `packages: read`, `pull-requests: none`, and the exact job ID `copilot-setup-steps`. Put the workflow in `.github/workflows/copilot-setup-steps.yml` on the default branch. Give the repository **Read** access under the private package’s **Manage Actions access** settings. Pass the workflow token to npm only on the install step.

```yaml
name: Copilot Setup Steps

on:
  workflow_dispatch:
  pull_request:
    paths:
      - .github/workflows/copilot-setup-steps.yml

permissions:
  contents: read
  packages: read
  pull-requests: none

jobs:
  copilot-setup-steps:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-node@v4
        with:
          node-version: '22'
          registry-url: https://npm.pkg.github.com
      - run: npm ci
        env:
          NODE_AUTH_TOKEN: ${{ secrets.GITHUB_TOKEN }}
```

If the repository uses both npmjs.org and GitHub Packages, configure the organization’s npm scope for the GitHub Packages registry so public packages still resolve from npmjs.org.

[GitHub Docs · Private dependencies for cloud agent](https://docs.github.com/en/copilot/tutorials/cloud-agent/give-access-to-resources) · [GitHub Packages · npm registry](https://docs.github.com/en/packages/working-with-a-github-packages-registry/working-with-the-npm-registry)

### Hooks, human approval, and agent handoffs

| Topic | What to remember |
| --- | --- |
| **Block a tool call** | Write a JSON object to stdout from `preToolUse`. Set `permissionDecision` to `deny` and include `permissionDecisionReason`. |
| **Repository hook location** | Put repository hook configuration under `.github/hooks/*.json`. Remember **hooks → JSON**, Actions workflows → YAML. |
| **Block a direct shell push in cloud agent** | Use `preToolUse` to evaluate and block the proposed command before it runs. In cloud agent, a hook decision of `ask` is treated as `deny`. |
| **Copilot PR checks haven't started** | Look for **Approve and run workflows**. With the default approval policy, a reviewer with write access reviews the changes and starts the workflows. |
| **Empty hook output** | Default behavior continues if no other hook denies the call. Empty output leaves normal permission checks in effect. |
| **Matcher `bash\|powershell`** | Where supported, a hook matcher is a full-match regex for the event-specific value. `bash\|powershell` matches either complete name. |
| **What does a PR plan check enforce?** | A PR workflow can check the plan file and the headings it tests. Instructions guide Copilot, and a required status check can block merge. Neither proves the plan was written before the first edit. |
| **Review a plan before implementation** | Define acceptance criteria, constraints, and allowed files → request a plan-only response → review scope, risks, and tests → record approval or requested changes in the issue. |
| **Why did the reviewer fail the check?** | The implementer contradicted the planner’s handoff. The plan ruled out schema changes, but the diff added a migration. Successful jobs can still produce an outcome that violates the approved scope. |
| **Reconstruct a multi-agent decision** | Define required decision fields → capture each agent outcome as it completes → write the final coordination summary to `$GITHUB_STEP_SUMMARY` → upload raw handoffs as an artifact. |
| **Select only the tools required for a pilot** | Inventory repo actions, checks, and external data needs → configure the agent’s tool allowlist → install build and test dependencies in Copilot setup steps → run a pilot issue and inspect the session output. |
| **Turn a vague checkout task into an agent-ready issue** | Identify affected paths, constraints, and allowed tools → define expected outputs → specify evidence such as checks, tests, and logs → create the GitHub issue. |
| **Resume from approved state** | The saved state SHA matches the current PR head. Exclude previous failed-run artifacts and rejected local notes. Resume with the approved plan, current diff, failed required checks, and runbook. |
| **Match escalation signals to responses** | Failed required check → review before marking ready. Denied shell command → use the approval path with the blocked command and its context. Missing MCP tools → inspect server startup and setup. Out-of-scope edits → narrow the issue and request review. |
| **Manual run started but the agent did not** | The approval gate blocked the agent-start job because the linked issue lacked the required `agent-approved` label. Inspect the gate result before investigating the agent itself. |
| **Protected agent-execution environment** | Required environment reviewers can hold the job before protected steps run and provide a human execution gate. A manual trigger records that someone started the workflow. Plan approval requires its own evidence. |
| **Migration PR fails a credential-sensitive test** | Keep the PR in draft and request owner review. This preserves the work and gives the responsible owner a chance to evaluate the test failure and credential request before advancing the PR. |
| **Payments refactor also changes workflow YAML** | Review the draft diff and require owner approval before merging. Passing unit tests and an approved issue cover different decisions from approving a workflow change. |
| **Prevent context drift across issue, PR, and CI** | Put scope and acceptance criteria in the issue → record decisions in the draft PR → upload validation evidence → review the durable records before marking ready. |
| **Find the first broken handoff** | Implementation touched the payment gateway outside the checkout retry scope. The payment test failure came next, and release review skipped because validation failed. Read the approved scope, changed paths, and job outcomes in that order. |

#### Example hook denial

```json
{
  "permissionDecision": "deny",
  "permissionDecisionReason": "Open a pull request instead of pushing directly."
}
```

[GitHub Docs · Hooks reference](https://docs.github.com/en/copilot/reference/hooks-reference)

#### Production rulesets and resume state

- **Merge requirements.** A ruleset can require a PR, two approvals, code-owner approval for covered paths, and required CI checks such as build, test, and CodeQL. Strict status checks require the PR branch to be up to date with its base. Passing CI still leaves review requirements in force.
- **Secret-backed execution.** For a job that references a protected environment, required reviewers can gate access to its environment secrets. A manual trigger and approval to run Copilot-generated workflows are separate from environment approval.
- **Path risk.** Documentation and issue templates can be classified as low risk, source and tests as medium risk, and auth, deploy, and compliance paths as high risk. Classification alone creates no review exemption. For overlapping patterns such as `src/**` and `src/auth/**`, define precedence in the policy implementation. These policy labels are illustrative rather than native GitHub ruleset keys.
- **Resume paused work.** Carry forward accepted architecture decisions, explicitly exclude abandoned approaches, and retest failures that apply to the current PR revision. Replace “continue from before” with the current commit, completed work, remaining task, accepted constraints, and the next validation target.

### Actions triggers, conditions, and concurrency

| Topic | What to remember |
| --- | --- |
| **Did the security job fail?** | `${{ needs.security.result == 'failure' }}` checks the named dependency. To let the notification job itself run after a dependency failure, use `${{ failure() && needs.security.result == 'failure' }}`. |
| **Validate PRs targeting main before merge** | Use `pull_request` with `branches: [main]`. The filter matches the PR’s target branch. |
| **Checkout `github.head_ref`** | It names the PR **source branch**. `github.base_ref` names the **target branch**. |
| **Keep the latest validation for the same PR branch** | Use workflow-level `concurrency` with a stable group such as `${{ github.workflow }}-${{ github.head_ref }}` and `cancel-in-progress: true`. `queue: max` cannot be combined with `cancel-in-progress: true`. Use the PR number when cancellation should be scoped to one PR. |
| **Required CI check in a merge queue** | Include `merge_group` in the CI triggers alongside `pull_request` as appropriate. The queue needs the check on its temporary merge-group commit. |
| **Release only on tags** | `${{ startsWith(github.ref, 'refs/tags/') }}`. `github.ref` holds the ref path. `github.sha` holds a commit ID. |
| **Chain with `workflow_run`** | Up to three downstream levels after the first workflow. `A → B → C → D` can run. This chain will not trigger `E`. |
| **Tell a PR comment from an issue comment** | In an `issue_comment` workflow, check `${{ github.event.issue.pull_request }}`. That property identifies a PR conversation comment. |
| **Preserve a manual input as Boolean** | Use `${{ inputs.perform_deploy }}`. `github.event.inputs` exposes values as strings. |
| **Fix only `release/2026.05`** | Select `release/2026.05` as the base branch when assigning the work. It supplies the starting point and PR target. |
| **Release-specific instructions and validation** | A PR into `release/2026.05` matches `pull_request.branches: ['release/**']`. Copilot code review reads repository guidance from the PR **head branch**. The branch filter controls workflow triggering. |
| **Release validation check stays pending** | The PR targets `main`, which does not match `release/**`. For `pull_request.branches`, inspect the base branch first. A required check can stay pending when its workflow is skipped by a filter. |
| **Dependency review trigger and permissions** | Include `.github/dependabot.yml` alongside manifests and lockfiles in the path filter. The workflow example uses `contents: read` and `pull-requests: read`. If the check is always required, design the workflow so unrelated changes can still report a result. |

#### Run a step when the PR has a `preview` label

```yaml
if: ${{ contains(github.event.pull_request.labels.*.name, 'preview') }}
```

- **Remember** · PR → labels → every name → contains `preview`.
- **Why it works** · `labels` is a list of objects. `*.name` pulls out each label’s name. `contains()` checks for an element in that list.
- **Common mistake** · `pull_request.label` treats the collection as one field. `github.ref` is a Git ref. Both miss the label list.
- **Trigger detail** · If adding the label itself should start a new run, include the `labeled` activity type under `pull_request.types`. The step condition is evaluated only after a workflow run starts.

[GitHub Docs · Expressions](https://docs.github.com/en/actions/reference/workflows-and-actions/expressions)

#### Cancel older validation runs for the same PR

Suppose **Validate PR** runs for PR **#42** from `feature/adaptive-limiter` into `main`.

| Context value | Example |
| --- | --- |
| `github.workflow` | `Validate PR` |
| `github.head_ref` | `feature/adaptive-limiter` |
| `github.event.pull_request.number` | `42` |

**Group by source branch**

```yaml
name: Validate PR
on: pull_request

concurrency:
  group: ${{ github.workflow }}-${{ github.head_ref }}
  cancel-in-progress: true
```

The group resolves to `Validate PR-feature/adaptive-limiter`. A new commit to the same PR triggers a run in the same group and cancels the older in-progress run. A PR from `feature/retry-policy` gets its own group.

**Group by PR number**

```yaml
concurrency:
  group: ${{ github.workflow }}-pr-${{ github.event.pull_request.number }}
  cancel-in-progress: true
```

PR **#42** resolves to `Validate PR-pr-42` and PR **#43** resolves to `Validate PR-pr-43`. Use the PR number to scope cancellation to one PR, including when two PRs share a source branch name. Omit `queue: max` when using `cancel-in-progress: true`.

| Group key | What it groups |
| --- | --- |
| `github.event.pull_request.number` | All pushes to the same PR. Different PRs get different groups. |
| `github.run_id` | Each workflow run has its own ID. New runs get separate groups. |
| `github.run_attempt` | The attempt number starts at 1 for each run. Unrelated first attempts can collide. |
| `runner.os` | The runner’s OS identifies a platform and is unavailable at workflow-level concurrency. |

**Remember** · Group by what should replace earlier work. Here that’s the **PR**. Using the PR number also keeps unrelated PRs with the same source branch name in separate groups.

[GitHub Docs · Concurrency](https://docs.github.com/en/actions/how-tos/write-workflows/choose-when-workflows-run/control-workflow-concurrency)

[GitHub Docs · Workflow events](https://docs.github.com/en/actions/reference/workflows-and-actions/events-that-trigger-workflows) · [Actions contexts](https://docs.github.com/en/actions/reference/workflows-and-actions/contexts)

### Jobs, matrices, and workflow reuse

| Topic | What to remember |
| --- | --- |
| **20 matrix jobs, four at once** | Set `strategy.max-parallel: 4`. This caps simultaneous jobs. |
| **Invalid key with job-level `uses`** | `runs-on`. The called workflow defines its own jobs and runners. |
| **Matrix from discovery JSON** | Set `strategy.matrix` to `${{ fromJSON(needs.discover.outputs.matrix) }}` and declare `needs: discover`. Parse the entire object containing `include`. |
| **`LOG_LEVEL` at three scopes** | The step-level value wins inside that step. Precedence is step → job → workflow. |
| **Allow only the experimental runtime to fail** | Set job-level `continue-on-error: ${{ matrix.experimental }}` with `experimental: true` only for the experimental entry. `fail-fast` controls cancellation of other matrix jobs after a mandatory failure. |
| **Write an `image_tag` step output** | Append `image_tag=value` to `$GITHUB_OUTPUT`. With step ID `meta`, read it as `steps.meta.outputs.image_tag`. |
| **Stop a hung test after 20 minutes** | Set `timeout-minutes: 20` on the test step. Keep the default failure behavior by leaving `continue-on-error` unset. |
| **Run job steps in Docker** | Use `jobs.<id>.container.image: node:22` as the nested configuration path. `services` configures sidecars. |
| **Composite action `run` step** | Set an explicit `shell` on each `run` step. The caller job supplies the runner. |
| **Default shell inside a job container** | `sh`. Specify `shell: bash` when you need Bash features and the image provides Bash. |

#### Reusable workflow or composite action?

| Cue | Reusable workflow | Composite action |
| --- | --- | --- |
| What it packages | One or more jobs | Multiple steps |
| Definition | Workflow YAML with `on: workflow_call` | `action.yml` with `runs.using: composite` |
| Call site | `jobs.<job_id>.uses` | `steps[*].uses` |
| Runner | Each called job selects its runner | Uses the enclosing job’s runner |

**Remember** · Workflow goes in a job slot. Composite goes in a step slot. Both use `uses`, so check its indentation.

[GitHub Docs · Reusing workflow configurations](https://docs.github.com/en/actions/concepts/workflows-and-actions/reusing-workflow-configurations)

### Deployments, permissions, and secure scripting

| Topic | What to remember |
| --- | --- |
| **Protected production deployment** | Set `environment: production` on the deployment job. The environment supplies its configured protection rules and secrets. |
| **Binary build provenance** | Set `contents: read`, `id-token: write`, and `attestations: write` under `permissions`. Read source, request an OIDC token, create the attestation. |
| **What `id-token: write` grants** | The job can request a GitHub OIDC token. The cloud provider's trust policy controls token exchange, and the resulting role controls cloud access. |
| **Pass an untrusted PR title to a shell** | Assign the PR title to a step-level `env` variable. Read that variable in quotes and use a fixed `printf` format. Direct expression interpolation in `run` inserts the value into the script source. |
| **Prevent administrator bypass in production** | Disable **Allow administrators to bypass configured protection rules** in the environment’s settings. |
| **Staging secrets and variables without deployment record** | Use environment object syntax with `name: staging` and `deployment: false`. |
| **Manual rollback to a known-good SHA** | `workflow_dispatch` collects the SHA, checkout selects it, and the `staging` environment can require approval. With only `contents: read` and `deployments: write`, `gh issue create` still lacks `issues: write`. Pass the SHA to shell commands via a quoted environment variable. |
| **Post a PR evaluation comment** | Use `contents: read` for checkout and `pull-requests: write` for the PR timeline comment. Supply `PR_NUMBER` and `GH_TOKEN` to `gh pr comment`. The comments endpoint also accepts `issues: write` as an alternative. |
| **Manual protected production release** | **Configuration** · `workflow_dispatch`, `production`, `./scripts/deploy_checkout.sh`. The environment controls approval and grants access to its secrets after protection rules pass. A check such as `test -f .github/releases/approved-release.json` confirms the record exists. |
| **Comment only on guardrail failure** | **Configuration** · `pull_request`, `pull-requests`, `run`. Use `contents: read` and `pull-requests: write` with `gh pr comment`. A plain `if: failure()` catches other earlier step failures. For a comment tied to the guardrail specifically, give that step `id: guardrail` and use `if: failure() && steps.guardrail.outcome == 'failure'`. If the script can fail for reasons other than a violation, have it report a separate violation signal. |

#### Caller and deployment environment define the same secret

The deployment job uses the environment’s `deployment_token`.

- **Remember** · Same name? The called job’s environment wins.
- **Why it works** · The reusable workflow’s deployment job sets `environment: production`. If that environment defines `deployment_token`, that job's `secrets.deployment_token` resolves to the environment value.
- **Common mistake** · Passing a secret explicitly from the caller can look like an override. The same-named environment secret still takes precedence for the job that references that environment.

[GitHub Docs · Reuse workflows](https://docs.github.com/en/actions/how-tos/reuse-automations/reuse-workflows)

#### Pass an untrusted PR title as shell data

```yaml
- name: Print PR title
  env:
    PR_TITLE: ${{ github.event.pull_request.title }}
  run: printf '%s\n' "$PR_TITLE"
```

#### Use staging secrets without a deployment record

```yaml
environment:
  name: staging
  deployment: false
```

[GitHub Docs · Secure use](https://docs.github.com/en/actions/reference/security/secure-use) · [Deployment environments](https://docs.github.com/en/actions/how-tos/write-workflows/choose-what-workflows-do/deploy-to-environment)

### Artifacts, diagnostics, and workflow commands

| Topic | What to remember |
| --- | --- |
| **File and line error annotation** | `echo "::error file=src/app.js,line=12::Invalid configuration"`. The properties identify the location and the text after the second `::` is the message. |
| **Collapsible log section** | Print `::group::A heading` before the lines and `::endgroup::` after them. |
| **Print untrusted text that contains command markers** | Print `::stop-commands::<random-token>`, print the text, then print `::<random-token>::` to re-enable processing. Use a fresh, unpredictable token for each run. Use this for workflow-command output and a quoted environment variable for shell input. |
| **Mask a runtime-generated secret** | Print `::add-mask::<value>` before the value appears in logs. Masking applies to subsequent output. |
| **Preserve diagnostics when tests fail** | **Configuration** · `pull_request`, `always()`, `actions/upload-artifact@v4`. Run validation for PRs into `main` and upload whatever diagnostics exist even after a failed step. If the plan copy follows the test command inside the same script, move the copy before the test, or into its own failure-tolerant step, so a failed test cannot skip it. |

#### Where an uploaded artifact lives

GitHub Actions artifact storage associated with the workflow run.

- **Remember** · A runner creates the file. `actions/upload-artifact` uploads it to the workflow run.
- **Common mistake** · Uploading a report or build output does not create a Git commit. The uploaded copy is available after the job ends, subject to retention.
- **Where to find it** · Open the workflow run’s summary and look under **Artifacts**.

[GitHub Docs · Workflow artifacts](https://docs.github.com/en/actions/concepts/workflows-and-actions/workflow-artifacts)

