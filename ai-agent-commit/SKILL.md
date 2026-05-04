---
name: ai-agent-commit
description: 'Draft or perform AI-generated git commits that comply with the repository commit policy. Use when writing commit messages, checking whether the current branch is safe to commit to, blocking commits to main or other protected branches, or preparing AI changes for PR review.'
argument-hint: 'Optional: commit scope or short change summary'
---

# AI Agent Commit

## What This Skill Produces

This skill produces either:

- A policy-compliant commit message for AI-authored changes.
- A completed non-interactive git commit on a safe feature branch when the user explicitly asks you to commit.

## When to Use

- The user asks you to commit changes.
- The user wants a commit message for AI-authored work.
- You need to verify whether the current branch is safe for committing.
- You need to package changes so they can go through PR review.

## Branch Safety Rules

- Never commit directly to `main`.
- Never commit directly to any other branch the repository or user has identified as protected.
- All AI-generated changes must land on a feature branch and go through PR review.
- If the current branch might be protected and you cannot verify that it is safe, stop and ask before committing.

## Commit Message Format

Use this exact structure:

```md
<type>(<scope>): <short summary>

Context:

- <bullet 1>
- <bullet 2>

What changed:

- <bullet 1>
- <bullet 2>

Risk level: Low | Medium | High
```

Keep the section titles and order exactly as shown.

## Commit Construction Rules

- Choose the narrowest meaningful `scope`, usually the affected feature, component, or module.
- Keep the summary short, specific, and grounded in the actual diff.
- Use exactly two bullets under `Context` unless the user explicitly asks for a different structure.
- Use exactly two bullets under `What changed` unless the user explicitly asks for a different structure.
- Set `Risk level` to `Low`, `Medium`, or `High` based on likely regression impact.
- If you are unsure about `type` or `scope`, infer the best fit from the changed files and ask only if the ambiguity materially affects the message.

### Suggested Commit Types

| Type       | Use when                                         |
| ---------- | ------------------------------------------------ |
| `feat`     | Adds user-visible behavior or capability         |
| `fix`      | Corrects a bug or broken flow                    |
| `refactor` | Restructures code without changing behavior      |
| `test`     | Adds or updates tests                            |
| `docs`     | Changes documentation only                       |
| `chore`    | Maintenance work with no product behavior change |
| `ci`       | Changes automation, pipelines, or release flow   |

## Procedure

1. Inspect the current branch and working tree.
2. If the current branch is `main`, stop. Do not commit.
3. If the current branch matches another known protected branch, stop. Do not commit.
4. Review the relevant diff so the commit reflects the actual changes.
5. Decide whether the user wants:
   - a commit message only, or
   - an actual commit.
6. Choose the commit `type`, `scope`, short summary, two `Context` bullets, two `What changed` bullets, and `Risk level`.
7. Format the message exactly as required.
8. If the user asked for message only, return the commit message and note any branch-safety concern.
9. If the user asked for an actual commit:
   - confirm the branch is safe,
   - stage only the intended files,
   - run a non-interactive git commit command,
   - remind the user that the next step is PR review.

## Decision Points

- If the branch is blocked, do not commit. Offer to help create or switch to a feature branch if the user wants that.
- If the change set contains unrelated edits, separate the commit scope or ask the user which files belong together.
- If the diff is too large for a single coherent commit, propose splitting it before committing.
- If the user requests an amend or history rewrite, perform it only when they explicitly ask for it.

## Completion Checks

- The branch is not `main` and not another known protected branch.
- The header matches `<type>(<scope>): <short summary>`.
- `Context:` appears exactly once with two bullets.
- `What changed:` appears exactly once with two bullets.
- `Risk level:` is present and set to `Low`, `Medium`, or `High`.
- If a commit was created, it used a non-interactive git command.
- If a commit was created, the user is reminded that the change still needs PR review.

## Example Output

`````md
fix(upload-file): handle empty upload response

Context:

- Branch staff could submit a file even when the API returned an empty payload.
- The upload flow needed a safer fallback to avoid runtime errors during customer onboarding.

What changed:

- Added a guard for empty upload responses before the component reads file metadata.
- Updated the related tests to cover the empty-response path and the successful upload path.

Risk level: Low

````---
name: ai-agent-commit
description: Prepare and, when explicitly requested, create Git commits that follow this repository's branch safety rules and structured commit message format. Use when drafting commit messages, summarizing a change set for review, or committing changes from a feature branch.
argument-hint: 'Optional: intended scope or short change summary, for example buy-order rounding fix'
---

# AI Agent Commit Skill

This skill standardizes AI-authored commits in this repository. It enforces branch safety, keeps commit scopes coherent, and produces commit bodies that explain the problem, the concrete implementation changes, and the expected risk level in a format that is easy to review in a pull request.

## When to Use This Skill

Use this skill when:

- The user asks for a commit message
- The user asks the agent to create a commit
- The user wants staged changes summarized in commit-ready form
- The agent needs to prepare a review-friendly commit body for a pull request workflow

## 1. Safety Rules

| Rule | Requirement |
| --- | --- |
| **Protected branches** | Never commit directly to `main` or any branch treated as protected by the repository |
| **Branching model** | All AI changes must go through a feature branch and pull request review |
| **Commit scope** | Commit only the files relevant to the requested change set |
| **Source of truth** | Derive commit text from the actual diff, staged changes, and verified context only |
| **History changes** | Do not amend, rebase, squash, or force-push unless the user explicitly asks |

If the current branch is `main` or another protected branch, stop before committing and tell the user that the changes must be committed from a feature branch.

## 2. Commit Header Format

The commit subject must use this format:

```text
<type>(<scope>): <short summary>
````
`````

````

### Header rules

- Use a conventional type such as `fix`, `feat`, `refactor`, `test`, `docs`, `chore`, `perf`, or `build`
- Use a narrow scope that matches the feature, module, or subsystem, such as `buy-order`, `login`, or `customer-form`
- Keep the summary concise, specific, and outcome-focused
- Do not end the summary with a period
- Prefer summaries that fit comfortably on one line

## 3. Commit Body Format

Use this exact structure for the body:

```text
Context:
- <bullet 1>
- <bullet 2>

What changed:
- <bullet 1>
- <bullet 2>

Risk level: Low | Medium | High
```

### Body rules

- `Context:` and `What changed:` must appear as standalone labels, not list items
- Each item under those sections must use `- ` bullet formatting
- Leave a blank line between the subject, each section, and the risk line
- Keep `Context` focused on the problem, prior behavior, or reason for the change
- Keep `What changed` focused on concrete implementation changes, tests, and affected code paths
- Use `Risk level:` as a single plain line at the end

## 4. Writing Guidance

### Context bullets should explain

- What was wrong, inconsistent, missing, or risky before the change
- Why the change was needed now
- Any user-visible bug, review concern, or architectural inconsistency being addressed

### What changed bullets should explain

- The concrete code paths, modules, services, or utilities that changed
- Whether logic was extracted, centralized, removed, or renamed
- Whether tests were added or updated, but only if that actually happened
- Any precision, validation, behavior, or data-flow changes introduced by the fix

### Avoid in commit bodies

- Generic bullets such as `updated code` or `fixed issue`
- Claims that are not supported by the diff
- Unrelated implementation detail that does not help a reviewer understand the change
- Mixing problem statements into `What changed` when they belong in `Context`

## 5. Risk Level Guidance

Choose exactly one of these values:

- `Low` for isolated changes with limited blast radius and straightforward review impact
- `Medium` for changes that affect multiple modules, shared behavior, or non-trivial runtime paths
- `High` for changes involving payments, authentication, critical data flow, migrations, or broad behavior changes

If the risk is not obviously low, justify the higher level from the actual change set rather than defaulting optimistically.

## 6. Workflow

Follow this sequence when using the skill:

1. Check the current branch before committing anything
2. Stop if the branch is `main` or otherwise protected
3. Review the relevant diff or staged files and group only coherent changes together
4. Draft the subject line using the narrowest useful scope
5. Write `Context` bullets that explain the reason for the change
6. Write `What changed` bullets that describe the actual implementation work
7. Set the risk level based on blast radius and operational sensitivity
8. If the user explicitly asked for a commit, create it only on a feature branch
9. Leave the change for pull request review rather than bypassing the branch policy

If the user asked only for a commit message, return the composed message without creating a commit.

## 7. Output Template

```text
<type>(<scope>): <short summary>

Context:
- <problem or prior behavior>
- <reason this needed to change>

What changed:
- <concrete implementation change>
- <concrete implementation change>

Risk level: <Low | Medium | High>
```

## 8. Example

```text
fix(form-handling): show a clear error when required input is missing

Context:
- Users could run into a confusing failure when submitting incomplete information.
- The previous behavior did not make it clear what needed to be corrected.

What changed:
- Added an early validation check before the main processing flow runs.
- Returned a clearer error message so the behavior is easier to understand and review.

Risk level: Low
```

## 9. Completion Standard

The task is complete only when:

- The commit message matches the required subject format
- The body uses the exact `Context`, `What changed`, and `Risk level` structure
- The content is grounded in the actual diff
- No commit is created on `main` or another protected branch
- The resulting change is ready for normal pull request review
````
