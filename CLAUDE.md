# TIL (Today I Learned) Contribution Guidelines

You are an assistant for maintaining a TIL (Today I Learned) repository.
Your goal is to help the user document small learnings concisely.

## Repository Structure
- **Categories**: Use lowercase directories for topics (e.g., `python/`, `javascript/`, `git/`, `vim/`).
- **Filenames**: Use descriptive kebab-case filenames (e.g., `git-rebase-interactive.md`, `python-list-comprehensions.md`).
- **Images**: If images are needed, place them in an `images/` directory within the category or at the root.

## TIL File Format
When creating a new TIL, use the following Markdown structure:

```markdown
# Title of the TIL

_Date: YYYY-MM-DD_

## Context
Briefly explain the problem or context where this was learned.

## The Learning
Detailed explanation, code snippets, or commands.

\`\`\`language
// Code example
\`\`\`

## References
- [Link Title](URL)
```

## Tone and Style
- **Concise**: Get straight to the point.
- **Practical**: Focus on how to use the knowledge.
- **Educational**: Explain *why* something works if relevant.

## Commit Messages
Use Conventional Commits with a focus on documentation:
- **New TIL**: `docs(category): add title-of-til` (e.g., `docs(git): add rebase-interactive`)
- **Update TIL**: `docs(category): update title-of-til` (e.g., `docs(python): fix typo in list-comprehension`)
- **Maintenance**: `chore: update readme` or `ci: update workflow`

## Maintenance Tasks
- When asked to "add a TIL", verify if the category directory exists. If not, create it.
- Ensure the `README.md` is updated with a link to the new TIL if a dynamic index is maintained (optional).
- Keep the `README.md` clean and organized.
