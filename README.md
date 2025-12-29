# GitHub Copilot Orchestra

> **A multi-agent orchestration system for structured, test-driven software development with AI assistance**

## What is GitHub Copilot Orchestra?

The "GitHub Copilot Orchestra" pattern transformed how I build with AI agents. Instead of juggling context and constantly switching modes, the Orchestra pattern provides a structured workflow that coordinates specialized AI subagents through a complete AI development cycle for adding a feature or making a change: planning → implementation → review → commit.

The system solves a critical challenge in AI-assisted development: maintaining code quality and test coverage while moving quickly. By enforcing Test-Driven Development (TDD) conventions and implementing quality gates at every phase, you get the speed of AI coding combined with best practices in software engineering.

## Key Features

- **🎭 Multi-Agent Workflow** - Conductor agent orchestrates specialized Planning, Implementation, and Code Review subagents, each optimized for their specific role.
- **✅ TDD Enforcement** - Strict Test Driven Development: writing failing tests, seeing them fail, writing minimal code to pass, and verifying success before proceeding.
- **🔍 Quality Gates** - Automated code review after each phase ensures standards are met before moving forward.
- **📋 Documentation Trail** - Comprehensive plan files and phase completion records create an audit trail for reviewing all work completed.
- **⏸️ Mandatory Pause Points** - Built-in stops for plan approval and phase commits keep you in control of the development process.
- **🔄 Iterative Cycles** - Each implementation phase follows the complete cycle: implement → review → commit before proceeding to the next phase.
- **💎 Keeps Context Concise** - The majority of the work is done in dedicated subagents, each with its own context window and dedciated prompt. This helps reduce hallucinations as the context window fills up.

## Architecture Overview

The Orchestra system provides **two architecture options**:

### Option 1: Root Agent with Skills (Recommended)
Uses a root agent (`Conductor.agent.md`) that references agent skills in `.github/skills/`. This is the recommended approach for most use cases.

**Components:**
- `Conductor.agent.md` - Main orchestration agent in the root directory
- `.github/skills/planning/SKILL.md` - Planning skill
- `.github/skills/implementation/SKILL.md` - Implementation skill  
- `.github/skills/code-review/SKILL.md` - Code review skill

### Option 2: Conductor Skill with Subskills
Uses the Conductor as a skill itself (`.github/skills/conductor/SKILL.md`) with subskill files in the same directory. This allows the Conductor to be invoked by other agents or workflows.

**Components:**
- `.github/skills/conductor/SKILL.md` - Main orchestration skill
- `.github/skills/conductor/planning.md` - Planning subskill
- `.github/skills/conductor/implementation.md` - Implementation subskill
- `.github/skills/conductor/code-review.md` - Code review subskill

---

### Conductor Agent/Skill
- Main orchestration that manages the complete development cycle.
    - Coordinates Planning, Implementation, and Code Review.
    - Generates the plan to be followed.
    - Handles user interactions and mandatory pause points.
    - Enforces the Planning → Implementation → Review → Commit cycle.
    - Uses Claude Sonnet 4.5 by default.

### Planning Skill
- Research and context gathering specialist.
    - Analyzes codebase structure and patterns.
    - Identifies relevant files and functions.
    - Returns structured findings to inform plan creation.

### Implementation Skill
- Implementation specialist following TDD conventions.
    - Executes individual phases of the development plan.
    - Writes failing tests first, then minimal code to pass.
    - Works autonomously within phase boundaries.

### Code Review Skill
- Quality assurance specialist.
    - Reviews uncommitted code changes using git to identify new code.
    - Validates test coverage and code quality.
    - Returns review results (`APPROVED/NEEDS_REVISION/FAILED`).

## Prerequisites

Before using the GitHub Copilot Orchestra, ensure you have:

- **VS Code Insiders** - Required for the custom chat modes feature that enables subagents and handing tasks off to them.
    - Download from: https://code.visualstudio.com/insiders/

- **GitHub Copilot Subscription** - Active subscription required for AI-powered agents
    - Individual or Business plan
    - GitHub Copilot Chat extension installed and enabled

- **Git** - Version control is integral to the workflow
    - Used for commit workflow at end of each phase
    - Recommended: Basic familiarity with git commands

## Installation

### Initial Setup

1. **Clone or Download the Repository**
   ```bash
   git clone https://github.com/ShepAlderson/copilot-orchestra.git
   cd copilot-orchestra
   ```
   
   Alternatively, download the repository as a ZIP file and extract it to your desired location or just copy the contents of the agent files from the browser.

2. **Verify Prerequisites**
    - Ensure the latest VSCode Insiders is installed and running.
    - Confirm the GitHub Copilot Chat extension is active (check the chat icon in the sidebar).
    - Verify your workspace is a git repository (run `git status` to confirm)
        - If not, you can use `git init` if you have git installed.

### Setup Custom Agents

The GitHub Copilot Orchestra uses custom chat modes and agent skills in VSCode Insiders. Choose the architecture that fits your needs:

#### Option 1: Root Agent with Skills (Recommended)

1. **Open VSCode Insiders** in your workspace directory
    ```bash
    cd /path/to/your/project
    code-insiders .
    ```

2. **Copy Files** - Copy the following to your project:
    - `Conductor.agent.md` to your project's root directory
    - `.github/skills/` directory (contains planning, implementation, and code-review skills)

3. **Install the Conductor agent**
    - **Project-scoped**: Keep `Conductor.agent.md` in your project's root directory
    - **User-scoped**: Copy to your User Data location:
        - Mac: `/Users/username/Library/Application Support/Code - Insiders/User/prompts`
        - Or use the manual setup process via the chat mode dropdown

#### Option 2: Conductor Skill with Subskills

1. **Copy the `.github/skills/conductor/` directory** to your project's `.github/skills/` folder

2. **The Conductor skill includes subskill files:**
    - `SKILL.md` - Main conductor skill
    - `planning.md` - Planning subskill
    - `implementation.md` - Implementation subskill
    - `code-review.md` - Code review subskill

3. **Invoke the skill** from another agent or workflow that references the conductor skill

### Create the Plans Directory

For both options, create the `plans/` directory (or the Conductor will make it when it writes out the first plan file):

```bash
mkdir plans
```

This directory will store:
- Core task plan documents (`<task-name>-plan.md`)
- Phase completion summaries (`<task-name>-phase-<N>-complete.md`)
- Final task completion summaries (`<task-name>-complete.md`)

**No Additional Configuration Required** - The agents/skills will appear in the GitHub Copilot Chat interface automatically.

## Using the Conductor Agent

Once setup is complete, you can start using the Conductor agent:

**Via Chat Mode Dropdown**:
- Open GitHub Copilot Chat
- Click the agent dropdown at the bottom of the chat panel
- Select "Conductor" from the list of available modes

## How It Works

The Conductor agent follows a strict four-stage cycle for every development task:

### 1. Planning Phase
- **User Request** - You describe what you want to build or change.
- **Delegates Research** - `Conductor` invokes the `planning-subagent` to gather comprehensive context about your codebase.
- **Plan Creation** - `Conductor` drafts a multi-phase plan (typically 3-10 phases) with specific objectives, files to modify, and tests to write.
- **Plan Approval** - `Conductor` stops, allowing you review and approve the plan before any implementation begins.
- **Plan Documentation** - Approved plan is saved to `plans/<task-name>-plan.md`.

### 2. Implementation Phase (repeated per plan phase)
- **Delegates Implementation** - `Conductor` invokes the `implement-subagent` with the specific phase objective and requirements.
- **TDD Execution** - `implement-subagent` follows strict Test-Driven Development:
    - Writes failing tests first.
    - Run tests to confirm they fail.
    - Writes minimal code to make the tests pass.
    - Run tests to verify they pass.
    - Apply linting and formatting.
- **Phase Summary** - `implement-subagent` reports completion back to the `Conductor`.

### 3. Review Phase (repeated per plan phase)
- **Quality Check** - `Conductor` invokes the `code-review-subagent` to validate the implementation.
- **Review Analysis** - `code-review-subagent` examines:
    - Test coverage and correctness.
    - Code quality and best practices.
    - Adherence to phase objectives.
- **Status Decision**:
    - Returns to `Conductor` with:
        - `APPROVED` - Proceed to commit step.
        - `NEEDS_REVISION` - Return to implementation with specific feedback.
        - `FAILED` - Stop and consult user for guidance.

### 4. Commit Phase (repeated per plan phase)
- **Phase Summary** - `Conductor` presents what was accomplished, to the user.
- **Documentation** - Phase completion file is saved to `plans/<task-name>-phase-<N>-complete.md`.
- **Commit Message** - `Conductor` generates a properly formatted git commit message.
- **MANDATORY STOP** - User makes the git commit and confirm readiness to continue.

**The Implement -> Review -> Commit cycle repeats** for each phase until the entire plan is complete, then the `Conductor` generates a final plan completion report.

### Agent Interaction Flow

```mermaid
sequenceDiagram
    participant User
    participant Conductor
    participant Planning
    participant Implement
    participant Review

    User->>Conductor: Request feature/change
    Conductor->>Planning: Gather context
    Planning-->>Conductor: Return findings
    Conductor->>User: Present plan
    User-->>Conductor: Approve plan
    
    loop For each phase
        Conductor->>Implement: Execute phase (TDD)
        Implement-->>Conductor: Report completion
        Conductor->>Review: Review implementation
        Review-->>Conductor: Return status
        
        alt Approved
            Conductor->>User: Present summary & commit message
            User-->>Conductor: Commit & continue
        else Needs Revision
            Conductor->>Implement: Revise with feedback
        else Failed
            Conductor->>User: Request guidance
        end
    end
    
    Conductor->>User: Plan complete
```

## Usage Example

Here's a realistic scenario demonstrating the complete workflow:

### Scenario: Adding User Authentication

**Initial Request:**
```
I need to add JWT-based user authentication to my Express API. 
Users should be able to register, login, and access protected routes.
```

**1. Planning Phase**
- `Conductor` delegates to `planning-subagent` to analyze your Express codebase.
- `planning-subagent` identifies existing patterns, middleware structure, and testing setup.
- `Conductor` creates a 5-phase plan:
    1. User model and database schema.
    2. Registration endpoint with validation.
    3. Login endpoint with JWT generation.
    4. Authentication middleware.
    5. Integration and end-to-end tests.

**2. You review and approve the plan**
- The `Conductor` comes back to the user with the draft of the plan. At the bottom of the draft of the plan it may have "Open Questions". Provide answers like:
    ```
    Answers to open questions

    1. Yes, use bcrypt for encrpyting passwords.
    2. ...
    ```

**3. Implementation -> Review -> Commit Cycle - Phase 1**
- `Conductor` invokes `implement-subagent` for "User model and database schema".
- `implement-subagent`:
    - Writes failing tests for User model (validation, password hashing, etc.).
    - Runs tests to see them fail.
    - Implements User model with minimal code.
    - Runs tests to verify they pass.
    - Applies linting/formatting.
- `Conductor` invokes `code-review-subagent`.
- `code-review-agent` returns `APPROVED`.
- `Conductor` presents summary and commit message to user:
    ```
    feat: Add User model with password hashing
    
    - Create User schema with email and password fields
    - Implement bcrypt password hashing on save
    - Add email validation and uniqueness constraint
    - Write comprehensive User model tests
    ```
- **You make the commit and tell `Conductor` to continue with "Proceed to next phase" in the chat.**

**4. Remaining Phases**
The cycle repeats for each remaining phase:
- Phase 2: Registration endpoint.
- Phase 3: Login endpoint.
- Phase 4: Auth middleware.
- Phase 5: Integration tests.

Each phase follows: **Implementation → Review → Commit** cycle.

**5. Completion**
- All phases complete.
- `Conductor` generates `plans/user-authentication-complete.md` with a full summary of what was accomplished.
- Your feature is fully tested, reviewed, and committed in logical increments.

## Generated Artifacts

The Orchestra system creates documentation files to track progress and provide an audit trail. You may want to add your `plans` directory to your `.gitignore` if you don't want to commit them, or you can commit them as a historical record. If you do this, maybe move the plans that have been completed to `plans/archived`, just to keep things tidy

### Plan File: `plans/<task-name>-plan.md`
Created after the user approves the plan. It contains:
- Task overview and objectives.
- Complete phase breakdown with steps.
- Suggestions of files and functions to create or modify.
- Tests to write.
- Open questions and decisions for the User to answer.
- Useful in case the User or the `Conductor` gets interrupted. You can always refer back to this and have the `Conductor` pick up where it left off.

**Example:** `plans/user-authentication-plan.md`

### Phase Completion: `plans/<task-name>-phase-<N>-complete.md`
Created after each phase commit, contains:
- Phase objective and summary.
- Files created/changed.
- Functions created/changed.
- Tests created/changed.
- Review status.
- Git commit message used.
- If you need to pick up the implementation cycle in the middle of a plan, due to interruption, you can tell the conductor to review the completed phase documents for context on where to start implementing again.

**Example:** `plans/user-authentication-phase-1-complete.md`

### Final Completion: `plans/<task-name>-complete.md`
Created when all phases are done, contains:
- Overall summary of the work completed.
- All phases completed (checklist).
- Complete list of files modified.
- Key functions/classes added.
- Test coverage summary.
- Recommendations for next steps.

**Example:** `plans/user-authentication-complete.md`

**Benefits (if committed with project):**
- **Audit Trail** - Full history of what was built and why.
- **Knowledge Transfer** - New team members can understand implementation decisions.
- **Project Documentation** - Natural documentation of feature development.
- **Review Process** - Easy to review what changed in each phase.

## Tips and Best Practices

### Working with the Conductor

- **Be Specific in Requests** - Provide context about your tech stack, existing patterns, and constraints.
    - Good: "Add JWT auth to my Express API using the existing PostgreSQL database. You can use the dev database connection string in the `.env-dev` file."
    - Less ideal: "Add authentication."

- **Review Plans Carefully** - The planning phase is your chance to guide the implementation.
    - Check that phases are appropriately scoped.
    - Verify test requirements align with your standards.
    - Ask questions about anything unclear.

- **Commit Frequently** - Don't skip the commit step between phases.
    - Each phase is designed to be independently committable.
    - Smaller commits are easier to review and revert if needed.
    - Creates a clear history of feature development.
    - The `code-review-agent` looks for uncommitted code as a basis for what to review.

### Maximizing Quality

- **Trust the TDD Process** - Testing first seems slow but catches issues early and provides clear guide rails for implementation via AI agent, even when those tests are written by the AI agent.
    - Failing tests confirm you're testing the right behavior.
    - Minimal code keeps implementations focused.
    - Passing tests give confidence to proceed.

- **Pay Attention to Reviews** - The `code-review-subagent` catches important issues
    - If status is `NEEDS_REVISION`, the feedback is handed back to the `Conductor` to start a new `implement-subagent` to fix the issue.
    - Use `FAILED` status as a signal to reassess approach. The `Conductor` will come back to the user and ask for input on what to do next.

- **Leverage the Documentation** - Phase completion files are valuable artifacts.
    - Review them before making commits.
    - Use them for PR descriptions and discussions.

### Optimizing Performance

- **Keep Phases Focused** - Smaller phases complete faster and with fewer iterations.
    - If a phase seems too large, ask Conductor to break it down.
    - Target 1-3 files modified per phase when possible.

- **Provide Good Context** - Help the `planning-subagent` find relevant code
    - Mention specific files or directories if you know them and attach them as explicit context in the AI chat.
    - Reference existing patterns to follow.
    - Call out any constraints or requirements upfront.

- **Use the Right Model** - The default agent configurations are optimized for a cost/quality balance.
    - Planning: Claude Sonnet 4.5 (project overview and collecting data for the plan)
    - Implementation: Claude Haiku 4.5 (efficient implementation of tests and code)
    - Review: Claude Sonnet 4.5 (thorough analysis and code review)
    - You can customize these in the `.agent.md` files if you'd like to use different models. Just change the model at the top of the file. (VSCode should autocomplete models available. Just delete past the `:` and type `:` again and a dropdown select should appear.)

## Working Across Multiple Sessions

The Orchestra pattern supports long-running development that spans multiple sessions. Here's how to maintain continuity:

### Resuming Interrupted Work

If a session is interrupted or you return later:

1. **Start Conductor** and say "continue" or "resume work on [task name]"
2. Conductor will:
   - Scan `plans/` for existing artifacts
   - Determine which phase you're on
   - Check for uncommitted changes
   - Run a smoke test
   - Resume from the correct point

### Progress Artifacts

Orchestra creates these files to enable cross-session continuity:

| File | Purpose |
|------|---------|
| `plans/<task>-plan.md` | Full plan with all phases |
| `plans/<task>-features.json` | Granular feature checklist (50-200+ items) |
| `plans/<task>-phase-N-complete.md` | Record of each completed phase |
| `plans/<task>-complete.md` | Final summary when done |

### Best Practices for Long-Running Work

- **Commit frequently** - Each phase should end with a commit
- **Don't stop mid-phase** - If you must stop, let Conductor document the state
- **Trust the artifacts** - Conductor reads plan files to determine where to resume
- **Keep tests passing** - Conductor runs smoke tests on resume to catch broken states

### Recovery from Broken State

If you resume and tests are failing:
1. Conductor will detect this and alert you
2. Fix existing bugs BEFORE starting new work
3. Alternatively, use `git stash` or `git checkout` to restore last good state

## Verification Modes

During planning, the Conductor will ask you to choose a verification mode:

### E2E Verification (Recommended for user-facing features)
- Each feature must be verified end-to-end
- Browser automation for web UIs, actual API calls for backends
- Takes longer but catches integration bugs
- Best for: Web apps, APIs, CLI tools

### Standard Verification (Faster iteration)
- Unit tests and code review are sufficient
- Faster development cycles
- Best for: Libraries, internal tools, rapid prototyping

You can choose different modes for different tasks based on your needs.

## Extending GitHub Copilot Orchestra to fit your needs

### Customizing Agents

Each agent is defined in a `.agent.md` file that you can modify:

**Adjust AI Model:**
- Change to other models.
- Available models in VSCode Insiders, as of the Nov. 5th, 2025:
    - `Auto (copilot)`
    - `Claude Sonnet 4.5 (copilot)`
    - `Claude Haiku 4.5 (copilot)`
    - `Claude Sonnet 4 (copilot)`
    - `Claude Sonnet 3.5 (copilot)`
    - `GPT-5 (copilot)`
    - `GPT-5-Codex (Preview) (copilot)`
    - `GPT-5 mini (copilot)`
    - `GPT-4.1 (copilot)`
    - `GPT-4o (copilot)`
    - `Grok Code Fast 1 (copilot)`
    - `Gemini 2.5 Pro (copilot)`

**Modify Instructions:**
- Edit the main section to change agent behavior, add new rules, or enforce project-specific conventions.

**Add Custom Tools:**
- Agents have access to various tools (file operations, terminal commands, etc.). The tool availability is managed by VS Code but you can provide guidance in instructions. You can also add MCP servers, if you use any. (I recommend getting `context7` setup. It's mentioned in the subagent files, and you can add it to the `tools` in the subagent files once it's setup.)

### Creating New Subagents

You can create specialized subagents for your workflow:

1. **Create a new `.agent.md` file** (e.g., `database-migration-subagent.agent.md`).
2. **Define the agent's role and instructions** using existing agents as templates.
3. **Update Conductor** to invoke your new subagent where appropriate.
4. **Test the integration** with a sample task.

**Ideas for subagents:**
- **deployment-subagent** - Specialized in deployment configurations.
- **security-audit-subagent** - Focused on security analysis.
- **performance-optimization-subagent** - Optimizes code performance.
- **documentation-subagent** - Generates comprehensive documentation.

## License

MIT License

Copyright (c) 2025 Shep Alderson

Permission is hereby granted, free of charge, to any person obtaining a copy
of this software and associated documentation files (the "Software"), to deal
in the Software without restriction, including without limitation the rights
to use, copy, modify, merge, publish, distribute, sublicense, and/or sell
copies of the Software, and to permit persons to whom the Software is
furnished to do so, subject to the following conditions:

The above copyright notice and this permission notice shall be included in all
copies or substantial portions of the Software.

THE SOFTWARE IS PROVIDED "AS IS", WITHOUT WARRANTY OF ANY KIND, EXPRESS OR
IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF MERCHANTABILITY,
FITNESS FOR A PARTICULAR PURPOSE AND NONINFRINGEMENT. IN NO EVENT SHALL THE
AUTHORS OR COPYRIGHT HOLDERS BE LIABLE FOR ANY CLAIM, DAMAGES OR OTHER
LIABILITY, WHETHER IN AN ACTION OF CONTRACT, TORT OR OTHERWISE, ARISING FROM,
OUT OF OR IN CONNECTION WITH THE SOFTWARE OR THE USE OR OTHER DEALINGS IN THE
SOFTWARE.

---
