# GitHub — Create and Initialize a Project

## Purpose

Create a new GitHub repository from a local directory based on a short project description. Initialize the repo, generate starter files, push to GitHub, and set up a feature branch workflow for PR-based development.

## Prompt

```
Create a new GitHub project from this directory.

1. Ask me for:
   - A short project name (used as the repo name).
   - A one-line description of what this project does.
   - Visibility: public or private.

2. Create the GitHub repository under my account using `gh repo create`.

3. Initialize the local directory as a git repo and link it to the new remote.

4. Generate starter files based on the project description:
   - `README.md` — Project name, description, and a basic structure outline.
   - `.gitignore` — Appropriate for the project type (infer from the description or ask if unclear).
   - Any other minimal scaffolding that fits the described project (e.g., a `src/` directory, a config file, a package.json). Keep it minimal — only add what's clearly needed.

5. Commit the starter files to `main` and push to GitHub.

6. Create a feature branch off `main` for subsequent work.

7. All future changes should be committed to the feature branch, pushed, and submitted as a PR for review before merging to `main`.
```

## Parameters

| Parameter          | Description                                      | Example                          |
| ------------------ | ------------------------------------------------ | -------------------------------- |
| `PROJECT_NAME`     | Repository name (lowercase, hyphens)             | `invoice-parser`                 |
| `DESCRIPTION`      | One-line summary of the project                  | `CLI tool to parse PDF invoices` |
| `VISIBILITY`       | Public or private                                | `private`                        |

## Reference

### Creating the repo via CLI

```bash
# Public repo
gh repo create <PROJECT_NAME> --public --description "<DESCRIPTION>" --source . --remote origin --push

# Private repo
gh repo create <PROJECT_NAME> --private --description "<DESCRIPTION>" --source . --remote origin --push
```

### Scaffolding guidelines

The starter files should be inferred from the project description. Examples:

| Description mentions       | Scaffolding to generate                                   |
| -------------------------- | --------------------------------------------------------- |
| Python / CLI / script      | `requirements.txt`, `src/main.py`, `.gitignore` (Python)  |
| Node / JavaScript / API    | `package.json`, `src/index.js`, `.gitignore` (Node)       |
| TypeScript                 | `package.json`, `tsconfig.json`, `src/index.ts`           |
| React / frontend           | Use `create-react-app` or `vite` scaffolding              |
| Documentation / prompts    | `README.md`, relevant `.md` files                         |
| Unknown / general          | `README.md`, `.gitignore`, empty `src/` directory         |

Keep scaffolding minimal. Do not over-generate — the user will build from here.

## Expected Outcome

1. New GitHub repository created under the user's account.
2. Local directory initialized as a git repo linked to the new remote.
3. Starter files generated based on the project description and committed to `main`.
4. Feature branch checked out and ready for development.
5. All future work follows the PR workflow.
