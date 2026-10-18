---

type: reference  
status: active  
created: 2026-10-18  
source: ""  
tags:

- ai
    
- llm
    
- coding-agents
    
- hermes
    
- ollama
    
- context-management
    
- tool-calling
    
- agent-safety
    

---
## Purpose

This note establishes a practical approach to selecting, evaluating and operating local AI coding agents.

The objective is to build agents that are **useful, predictable, observable and economical**, rather than simply granting a capable model unrestricted access to tools and hoping it behaves sensibly.

The principles apply to Hermes, OpenCode, Aider and future custom agent frameworks, although individual capabilities and configuration options vary.

## 1. What Makes an Agent Different?

An LLM generates responses. An agent uses an LLM within a system that can select actions, invoke tools, observe results and continue working towards an objective.

A simplified agent loop is:

```
User objective
      ↓
Agent receives context
      ↓
Model selects next action
      ↓
Tool executes action
      ↓
Result returned to agent
      ↓
Agent evaluates progress
      ↓
Continue or finish
```

The model is only one component.

Agent behaviour also depends on:

- Tool definitions and permissions
    
- Context construction and retention
    
- Execution-loop design
    
- Error handling and retries
    
- State management
    
- Budget enforcement
    
- Completion criteria
    

A capable model inside a poorly controlled agent can perform worse than a smaller model operating within a disciplined framework.

## 2. Lessons from the Hermes Experiment

Our initial Hermes experiments included system administration, a small camera-monitoring application and Discord configuration.

The agent consumed approximately **64 million tokens** during those activities.

The central problem was not simply the selected model.

The agent was operating with inadequate context management and insufficient control over its execution loop.

The large advertised context window of the model made it possible to accumulate substantial amounts of information, but did not make that information useful.

### Lessons

**A large context window is not a memory-management strategy.**

An agent can repeatedly reprocess tool output, source files and earlier reasoning without making meaningful progress.

**Autonomy requires boundaries.**

Without limits on tool calls, iterations and resource consumption, an agent can continue working long after the marginal benefit has disappeared.

**Progress must be measurable.**

A busy agent is not necessarily a productive agent.

The important measure is completed, verified work—not the number of actions performed.

## 3. Model Selection

Selecting a model for agent work differs from selecting one for conversational use.

A good coding-agent model needs more than fluent text generation.

|Capability|Why it matters|
|---|---|
|Instruction adherence|Respects scope and constraints|
|Tool-call reliability|Produces valid, appropriate tool requests|
|Code comprehension|Understands existing implementations|
|Repository navigation|Locates relevant files efficiently|
|Debugging|Investigates causes before proposing changes|
|Planning|Breaks work into manageable steps|
|Error recovery|Responds sensibly to failed operations|
|Completion judgement|Recognises when the task is finished|

Model size and benchmark rankings are useful indicators, but they are not substitutes for testing the model with the actual agent framework.

### Local hardware considerations

Our Windows AI workstation provides an RTX 5070 Ti with 16 GB VRAM.

For larger coding models, quantisation and context length must be balanced against available GPU memory.

A model that runs entirely on the GPU with a practical context window may provide a better interactive experience than a larger model requiring substantial CPU offloading.

See [[Quantisation, VRAM and Context]].

### Model evaluation principle

**Evaluate the complete agent system, not the model in isolation.**

The same model may perform differently when paired with different tool definitions, prompts, context-management policies and execution frameworks.

## 4. Context Management

Context management is one of the most important responsibilities of an agent framework.

An agent should retain information that helps it complete the task while avoiding unnecessary repetition.

### Useful context

- Current objective
    
- Relevant project instructions
    
- Files directly related to the task
    
- Important architectural decisions
    
- Recent tool results
    
- Known failures and their causes
    
- Current progress and remaining work
    

### Low-value context

- Repeated full-file reads
    
- Unfiltered command output
    
- Entire logs when only a few lines matter
    
- Duplicate search results
    
- Superseded plans
    
- Long histories of unsuccessful attempts
    
- Unrelated repository files
    

### Context discipline

An effective agent should:

1. Retrieve information selectively.
    
2. Avoid rereading unchanged material.
    
3. Summarise completed investigation stages.
    
4. Preserve important findings across context resets.
    
5. Discard irrelevant intermediate output.
    
6. Reintroduce only the information required for the next action.
    

### Persistent state versus conversation history

An agent should not depend entirely on its conversation history to remember project progress.

Durable information belongs in external state, such as:

- Project documentation
    
- Task records
    
- Decision logs
    
- Structured checkpoints
    
- Source control
    
- Test results
    

The conversation provides working context; persistent state provides continuity.

## 5. Token and Execution Budgets

An agent needs explicit resource boundaries.

Useful limits include:

|Limit|Purpose|
|---|---|
|Maximum tokens per task|Prevent excessive model consumption|
|Maximum tool calls|Restrict uncontrolled execution|
|Maximum iterations|Prevent endless reasoning/action loops|
|Maximum runtime|Bound elapsed execution time|
|Maximum retries|Prevent repeated failures|
|Maximum context size|Control memory and prompt-processing cost|
|Maximum tool-output size|Prevent logs and files overwhelming context|

Not every framework supports all these limits natively.

Where necessary, they must be enforced by an external controller.

### Budget exhaustion

Reaching a limit should not automatically be treated as failure.

A well-designed agent should stop cleanly and report:

- Work completed
    
- Current state
    
- Outstanding tasks
    
- Relevant errors
    
- Recommended next action
    

This allows the user or supervising process to decide whether further execution is justified.

### Avoid unlimited retries

Repeatedly attempting the same failing operation without changing the conditions is rarely productive.

A sensible retry policy should distinguish transient failures from persistent errors.

For example, a temporary network timeout may justify a retry.

A command repeatedly failing because of an invalid argument requires investigation, not more attempts.

## 6. Tool Permissions and Trust Levels

Agent autonomy should increase gradually as confidence grows.

For Hermes, our initial preference is to retain human approval for consequential operations.

A useful conceptual progression is:

|Level|Permissions|Typical use|
|---|---|---|
|1 — Observe|Read files, inspect state, gather information|Initial evaluation|
|2 — Propose|Produce plans, patches and commands without applying them|Assisted development|
|3 — Controlled execution|Make approved changes within a defined workspace|Supervised coding|
|4 — Bounded autonomy|Execute predefined classes of changes and tests|Trusted repeatable workflows|
|5 — Extended autonomy|Perform broader operations under explicit policy|Mature, thoroughly evaluated agents|

These are proposed operating levels, not native Hermes permission modes.

### High-impact operations

Require explicit approval for actions such as:

- Deleting or overwriting important files
    
- Changing system configuration
    
- Installing or removing software
    
- Modifying network or firewall settings
    
- Changing access permissions
    
- Accessing secrets or credentials
    
- Deploying changes to production services
    
- Performing destructive database operations
    

The exact approval policy should reflect the environment and task.

### Least privilege

An agent should receive only the tools and permissions needed for its current objective.

A repository-analysis task does not require unrestricted system administration.

A documentation task does not require permission to deploy containers.

Restricting available capabilities reduces both accidental damage and the complexity of agent decisions.

## 7. Checkpoints and Recovery

Long-running tasks should be divided into verifiable stages.

For coding work, useful checkpoints include:

1. Requirements understood.
    
2. Relevant files identified.
    
3. Existing behaviour documented.
    
4. Proposed changes reviewed.
    
5. Changes applied.
    
6. Tests executed.
    
7. Results verified.
    
8. Final summary produced.
    

### Source control

Git provides a practical foundation for controlled coding-agent work.

Before allowing an agent to modify a repository:

- Confirm the working tree state.
    
- Identify unrelated existing changes.
    
- Establish the intended branch.
    
- Define which files are in scope.
    
- Agree on validation requirements.
    

The agent should not silently overwrite unrelated user changes.

### Small changes are easier to trust

Prefer incremental, reviewable modifications over large batches of speculative edits.

A small patch with a clear test is easier to evaluate and revert than a broad rewrite.

## 8. Observability

An agent should make its actions understandable without requiring the user to read its entire conversation history.

Useful operational records include:

|Record|Purpose|
|---|---|
|Task identifier|Correlate actions with objectives|
|Model and version|Reproduce behaviour|
|Tool calls|Establish what was attempted|
|Tool results|Record observable outcomes|
|Files modified|Identify scope of changes|
|Token usage|Measure resource consumption|
|Runtime|Measure execution efficiency|
|Errors and retries|Identify loops and failure patterns|
|Checkpoints|Support recovery|
|Final verification|Establish whether the task succeeded|

### Logging considerations

Agent logs may contain sensitive information.

Avoid unnecessarily recording:

- API keys
    
- Authentication tokens
    
- Passwords
    
- Private documents
    
- Unredacted environment variables
    

Operational visibility should not come at the expense of credential security.

## 9. Detecting Unproductive Agent Behaviour

An agent can appear active while making little progress.

Warning signs include:

- Repeatedly reading the same files
    
- Reissuing equivalent searches
    
- Repeating failed commands
    
- Rewriting previously completed work
    
- Producing increasingly elaborate plans without execution
    
- Expanding the task beyond its original scope
    
- Consuming large amounts of context with little new information
    
- Continuing after acceptance criteria have been met
    

### A progress-based control loop

A supervising framework can periodically ask:

1. What changed since the previous checkpoint?
    
2. What evidence demonstrates progress?
    
3. What remains unresolved?
    
4. Is the next action materially different from previous failed attempts?
    
5. Does continuing justify the additional cost?
    

If the agent cannot demonstrate meaningful progress, it should stop, request guidance or change strategy within its permitted scope.

## 10. Evaluating Agent Performance

Token generation speed is not a sufficient performance metric.

A more useful evaluation considers:

**Correctness:** Did the agent produce the intended result?

**Efficiency:** How much time and computation were required?

**Reliability:** Does it succeed consistently across repeated tests?

**Safety:** Did it respect permissions and avoid unrelated changes?

**Maintainability:** Are the resulting changes understandable and appropriate?

### Representative benchmark

For the SillyTavern Prompt Workbench project, a useful initial task is:

> Inspect the SillyTavern source code and identify the request-assembly and outbound API call paths. Document the relevant files and functions, propose instrumentation points, and do not modify the repository.

This tests:

- Repository exploration
    
- Code comprehension
    
- Evidence-based reporting
    
- Scope adherence
    
- Tool efficiency
    
- Completion judgement
    

It also avoids granting write permissions during the initial evaluation.

### Benchmark record

|Property|Result|
|---|---|
|Agent framework||
|Model||
|Quantisation||
|Context window||
|Task||
|Tool calls||
|Tokens consumed||
|Elapsed time||
|Files changed||
|Tests passed||
|Human intervention||
|Outcome||

Compare results using equivalent tasks and permission policies.

## 11. Hermes — Initial Operating Strategy

Our agreed direction is to establish Hermes as a useful, trustworthy assistant before giving it greater control.

### Phase 1 — Establish a baseline

- Select a suitable local model.
    
- Confirm Ollama connectivity.
    
- Validate basic tool calling.
    
- Test read-only system inspection.
    
- Establish context and execution budgets.
    
- Record token usage and tool activity.
    

### Phase 2 — Controlled tasks

Introduce narrowly scoped activities such as:

- Reviewing configuration
    
- Analysing logs
    
- Inspecting repositories
    
- Producing change proposals
    
- Running approved diagnostic commands
    

Require approval before modifying system state.

### Phase 3 — Supervised execution

Once behaviour is reliable:

- Permit changes within designated workspaces.
    
- Require verification after changes.
    
- Use Git checkpoints where appropriate.
    
- Maintain clear limits on execution.
    
- Review failures and refine operating policies.
    

### Phase 4 — Custom agent framework

After gaining practical experience with Hermes, use those findings to design a custom framework.

Potential areas of development include:

- Explicit task state
    
- Context lifecycle management
    
- Tool permission policies
    
- Budget enforcement
    
- Checkpointing
    
- Progress evaluation
    
- Structured execution logs
    
- Recovery and resumption
    

The intention is to avoid building an entire framework before understanding which limitations actually matter in day-to-day use.

## 12. Design Principles for a Future Framework

### Separate the model from the controller

The model proposes actions.

The controller determines which actions are permitted, executes tools, tracks state and enforces limits.

### Keep state explicit

Important task information should be stored in a form that survives context resets and agent restarts.

### Make actions auditable

Every consequential operation should be attributable to a task and an explicit decision.

### Enforce budgets outside the model

A model should not be solely responsible for deciding whether it has consumed too many resources.

### Prefer bounded tasks

Define objectives and acceptance criteria before execution.

### Verify results independently

An agent reporting success is not proof that the task succeeded.

Use tests, command results, file inspection or other appropriate evidence.

### Support graceful interruption

Stopping an agent should preserve enough state to understand what happened and resume safely where appropriate.

## Related Notes

- [[Quantisation, VRAM and Context]]
    
- [[LLM Generation Parameters — A Practical Guide]]
    
- [[SillyTavern Prompt Construction and Presets]]
    
- [[SillyTavern Request and Response Pipeline]]
    
- [[SillyTavern Docker Patching and Maintenance]]
    

## Key Takeaway

**An effective coding agent is not simply a powerful LLM with access to a shell.**

Its reliability depends on context discipline, bounded execution, appropriate permissions, persistent state and verifiable outcomes.

Our approach is to make Hermes useful under supervision, learn from its strengths and limitations, and then apply those lessons to a custom agent framework designed around explicit control rather than unrestricted autonomy.