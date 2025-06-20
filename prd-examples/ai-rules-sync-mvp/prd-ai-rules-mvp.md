# PRD: AI Rule Sync (MVP)

## 1. Introduction/Overview

This document outlines the requirements for the Minimum Viable Product (MVP) of `ai-rule-sync`, a lightweight CLI tool for developers to manage and sync their AI assistant rule files across multiple projects.

The core problem this tool solves is the difficulty of keeping AI rule files (e.g., for Cursor, Claude) consistent across different Git repositories. Manually copying files leads to divergence and wasted effort. This tool provides a simple, command-driven way to manage these files using a central Git repository as the single source of truth.

The primary goal is to create a "sharp tool" for developers that is simple to use, non-intrusive, and empowers users who are comfortable with Git.

## 2. Goals

- **Zero-to-setup in <30s:** A user can initialize a project with shared rules quickly.
- **Single-command update:** A user can update their project's rules to the latest version with one command.
- **Safe publishing:** The tool will prevent users from accidentally overwriting their own unpublished local changes.
- **Frictionless install:** The tool can be installed via a simple script or common package managers.
- **Cross-platform:** The tool will work on macOS, Linux, and WSL2 (starting with macOS for the MVP).

## 3. User Stories

1.  **Bootstrap:** "As a dev starting a new microservice, I run `ai-rule-sync init` and get all my standard AI rules copied into my project in under 5 seconds."
2.  **Update:** "As a dev, when I update a rule in my central rules repo, I run `ai-rule-sync pull` in my project to get the latest version."
3.  **Publish:** "As a dev, after I tweak a rule file locally, I run `ai-rule-sync publish <file>` to push it back to my central rules repo."
4.  **Direct Edit:** "As a dev who wants to make several changes to my rules, I run `cd $(ai-rule-sync cd)` to navigate directly to my rules repo, where I can use standard Git commands."

## 4. Functional Requirements

1.  The tool **must** provide a command, `init`, to set up a project.
    - On its first ever run, this command will prompt the user for the Git URL of their rules repository.
    - It will store this URL in a global configuration file (`~/.ai-rule-sync/config.yaml`).
    - It will clone the user's repository into a local directory (`~/.ai-rule-sync/repo`).
    - It will copy all files from the local repo into the project's rules directory (default: `.cursor/rules`).
2.  The tool **must** provide a command, `pull`, to update the rules in a project.
    - It must first check for "drift" by comparing the project's current rule files against the versions stored in the local repo (`~/.ai-rule-sync/repo`).
    - If drift is detected, it must report the changes and exit without overwriting them.
    - If no drift is detected, it will fetch the latest changes from the remote origin into the local repo and copy them into the project.
3.  The tool **must** provide a command, `publish <file>`, to publish a changed rule file back to the origin.
    - This command will copy the specified file from the project to the local repo, commit the change, and push it to the remote `origin`.
4.  The tool **must** provide a command, `cd`, that prints the absolute path of the local rules repo (`~/.ai-rule-sync/repo`).

## 5. Non-Goals (Out of Scope)

- IDE plugins or GUI front-ends.
- Semantic versioning for rules. The `main` branch of the origin repo is considered the "latest" version.
- LLM-assisted merge conflicts.

## 6. Design Considerations

- **CLI Output:** All output from commands should be clear, concise, and informative. Successful operations should provide confirmation, and errors should be actionable. Use color and formatting to improve readability where appropriate, but do not rely on it exclusively.

## 7. Technical Considerations

- **Language:** The tool will be built in Go (version 1.22 or later) and distributed as a single, statically-linked binary.
- **Dependencies:** It will use `Cobra` for the CLI structure and `Viper` for configuration management.
- **Distribution:** Release automation will be handled by `GoReleaser` to publish binaries to GitHub Releases, with support for Homebrew and Scoop for installation.
- **Configuration:** All configuration and cached data will be stored within the `~/.ai-rule-sync/` directory to keep the user's home directory clean.

## 8. Success Metrics

- **Adoption:** The tool is successfully used to manage rules across at least 3 personal or team projects.
- **Workflow Improvement:** Users report that the tool is significantly easier and more reliable than manually copying files.

## 9. Open Questions

1.  Should the project be named `ai-rule-sync` or something more brandable?
2.  Should the tool offer to auto-install a Git pre-commit hook that runs `ai-rule-sync pull --summary`?
3.  What is the best way to manage rules for different AI models (e.g., Cursor, Claude) within a single origin repo?
