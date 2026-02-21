# Clone Repo and Initialize Claude

## Purpose

Clone an existing GitHub repository into a local directory and run Claude Code's `/init` command to generate a `CLAUDE.md` file with project context, conventions, and instructions.

## Prompt

```
Clone <REPO_URL> into <TARGET_DIRECTORY> and initialize it for Claude Code.

1. Clone the repository:
   - Run `git clone <REPO_URL> <TARGET_DIRECTORY>`.
   - Change into the cloned directory.

2. Explore the project:
   - Identify the language(s), framework(s), and build tools in use.
   - Note any existing documentation, CI config, or contribution guidelines.

3. Run `/init` to generate a `CLAUDE.md` file at the repo root.
   - The file should capture: project purpose, tech stack, build/test/lint commands, directory structure, coding conventions, and any repo-specific workflow notes.
   - Incorporate information from existing docs (README, CONTRIBUTING, CI configs) rather than duplicating them verbatim.

4. After `/init` completes, show me a summary of what was generated so I can review and refine it.
```

## Parameters

| Parameter          | Description                                      | Example                                            |
| ------------------ | ------------------------------------------------ | -------------------------------------------------- |
| `REPO_URL`         | HTTPS or SSH URL of the repository to clone      | `https://github.com/org/project.git`               |
| `TARGET_DIRECTORY` | Local path where the repo should be cloned       | `~/projects/project`                               |

## Expected Outcome

1. Repository cloned into the specified local directory.
2. `CLAUDE.md` generated at the repo root with project-specific context for Claude Code.
3. Summary of the generated file presented for user review.
