# PRD: Automated AI-Thoughts Digest

## 1. Introduction/Overview

This document outlines the requirements for the "AI-Thoughts Digest" project. The primary goal is to create a fully automated, self-hosted system that converts posts from a public Telegram channel into a weekly email digest. This digest will be distributed via Substack and cross-posted to a personal blog.

This PRD is intended for an AI agent and provides the necessary details to implement the core functionality.

## 2. Goals

- **Automate Content Aggregation:** Eliminate the manual process of collecting and formatting weekly content.
- **Consistent Delivery:** Ensure a new digest is reliably published every week on a schedule.
- **High-Quality Output:** Generate a well-formatted digest in both Markdown and HTML suitable for email.
- **Zero-Cost Infrastructure:** The entire pipeline must run on self-hosted infrastructure without incurring additional costs beyond Substack's platform fees.
- **Extensibility:** Build a modular system where new features (like LLM summaries) can be added later.

## 3. User Stories

- **As the project owner, I want to** automatically fetch all new posts from my Telegram channel, **so that** I don't have to manually copy-paste content.
- **As the project owner, I want to** have all posts from the past week compiled into a single document, **so that** I can review them before publishing.
- **As the project owner, I want to** automatically generate both Markdown and HTML versions of the digest, **so that** it's compatible with different platforms.
- **As the project owner, I want to** have the digest automatically uploaded as a draft to my Substack, **so that** I only need to perform a final review and click "publish".
- **As the project owner, I want the** entire process to run on a weekly schedule, **so that** the digest is published consistently without my intervention.
- **As the project owner, I want to** deploy updates to the application with a single command, **so that** I can easily maintain and improve the service.
- **As the project owner, I want to** have daily backups of my data, **so that** I can recover from a server failure or data corruption.

## 4. Functional Requirements

1.  **Configuration:** The system must load all necessary secrets and configuration (API keys, channel names, etc.) from a `.env` file.
2.  **Database:** A SQLite database must be used to store messages fetched from Telegram. The schema should include tables for messages and digests.
3.  **Ingestion:**
    1.  The system must connect to the Telegram API using `telethon`.
    2.  On first run, it must back-fill the database with the entire message history from the specified channel.
    3.  After back-filling, the system must listen for new messages and save them to the database in real-time.
4.  **Digest Generation:**
    1.  A script must be created to query the database for all messages posted in the last 7 days.
    2.  The script will compile these messages into a single document.
    3.  The output must be generated in both Markdown and HTML formats using `jinja2` templates.
    4.  The digest builder should handle basic formatting, like bullet points for posts and de-duplicating links.
5.  **LLM Summarization (Optional):** The system should optionally allow calling OpenAI's GPT-4o model to generate summaries. This will be gated by a feature flag and require an `OPENAI_API_KEY`.
6.  **Publishing:**
    1.  The system must be able to authenticate with Substack's unofficial GraphQL API using a session cookie.
    2.  It must create a new **draft** post on Substack.
    3.  The generated HTML content of the digest will be used as the body of the Substack post.
7.  **Scheduling:** The entire process of generating and publishing the digest must be scheduled to run weekly on a specific day and time (e.g., Sunday at 08:00), managed by `APScheduler`.
8.  **Deployment:** The application must be containerized using a `Dockerfile` and be deployable via Kamal. All secrets will be managed by 1Password and injected at runtime.
9.  **Error Handling:** On external API failures (e.g., Substack, Telegram), the system will log the error and exit gracefully without retrying.
10. **Exception Tracking:** All unhandled exceptions in the application will be captured and sent to Sentry for monitoring and alerting.
11. **Backups:** A daily backup of the application database and critical configuration files must be created and stored in a designated backup directory on the server.

## 5. Non-Goals (Out of Scope for MVP)

- **Complex Media Handling:** The initial version will focus on text-based content and links. Embedded images, videos, or audio files will not be included in the digest body, but may be represented as links.
- **Advanced Analytics:** The system will not track link clicks or open rates beyond what Substack provides. No custom analytics like `utm_source` will be added in the first version.
- **Cloud Deployment:** The solution will be designed for self-hosting. There will be no specific features for deploying to AWS, GCP, etc.
- **Web UI/Dashboard:** There will be no web-based user interface for managing the system. All operations will be performed via CLI scripts and cron jobs.
- **Direct Blog Publishing:** The system will only publish to Substack. The cross-posting to the personal blog will be handled by embedding Substack's provided RSS feed.
- **API Integration:** The publisher will interact with Substack's `/api/v1/graphql` endpoint. This is an unofficial API and may be subject to change.
- **Secret Management:** All secrets (e.g., registry credentials, API keys) will be stored in 1Password and loaded into the environment for deployment, e.g. via `op run`.

## 6. Design Considerations

- **Templating:** The digest's look and feel will be defined in `jinja2` templates. Separate templates should be created for the Markdown and HTML outputs.
- **CLI Interface:** Each main component (ingest, digest, publish) should be a runnable Python module with clear CLI arguments, as documented in the `README.md`.

## 7. Technical Considerations

- **Language/Runtime:** Python 3.12
- **Package Manager:** `uv`
- **Core Libraries:** `telethon`, `jinja2`, `markdown-it-py`, `APScheduler`, `SQLModel`, `pydantic`, `sentry-sdk`.
- **Development Tools:** `ruff`, `black`, `pytest`.
- **API Integration:** The publisher will interact with Substack's `/api/v1/graphql` endpoint. This is an unofficial API and may be subject to change.
- **Deployment:** The application will be deployed using Kamal to a server at `ubuntu@server01.example.com`.
- **Secrets Management:**
  - All secrets (`TELEGRAM_*`, `SUBSTACK_*`, `KAMAL_REGISTRY_PASSWORD`, `SENTRY_DSN`, `OPENAI_API_KEY`, etc.) will be stored in 1Password.
  - Deployment commands (`kamal deploy`, `kamal setup`) should be run within a 1Password-injected environment, for example: `op run -- kamal deploy`.
  - The `.kamal/env.secret` file will reference these environment variables.
- **Dockerfile:** A `Dockerfile` will be created to containerize the Python application, including installing dependencies from `requirements.txt`. The application itself will be run as a long-running process (e.g. the scheduler).
- **Container Registry:** Docker Hub will be used as the container registry. The image will be named `[DOCKERHUB_USERNAME]/ai-thoughts`.
- **Exception Monitoring:** Sentry will be used for tracking exceptions. The `SENTRY_DSN` will be configured via environment variables.
- **LLM Integration:** If enabled, the system will integrate with the OpenAI API using the `openai` library. It will require the `OPENAI_API_KEY` secret.

## 8. Success Metrics

- **Successful Automation:** The primary success metric is the system's ability to run for 4 consecutive weeks without manual intervention, successfully creating a draft post on Substack each week.
- **Time Saved:** Reduction in time spent creating the weekly digest from several hours to near-zero.
- **Code Quality:** Core logic achieves at least 85% unit test coverage.

## 9. Open Questions

None.

## 10. Deployment Strategy

The application will be deployed as a Docker container using Kamal.

- **Tool:** Kamal (`gem install kamal` or via Docker).
- **Target Server:** `ubuntu@server01.example.com`
- **Configuration (`config/deploy.yml`):**
  - `service`: `ai-thoughts`
  - `image`: `[DOCKERHUB_USERNAME]/ai-thoughts`
  - `servers`: `[ "ubuntu@server01.example.com" ]`
  - `builder`: Use default local docker builder.
  - `registry`: Credentials (for `[DOCKERHUB_USERNAME]` and a `KAMAL_REGISTRY_PASSWORD`) will be passed from environment variables sourced from 1Password.
- **Secrets Management:**
  - All secrets (`TELEGRAM_*`, `SUBSTACK_*`, `KAMAL_REGISTRY_PASSWORD`, `SENTRY_DSN`, `OPENAI_API_KEY`, etc.) will be stored in 1Password.
  - Deployment commands (`kamal deploy`, `kamal setup`) should be run within a 1Password-injected environment, for example: `op run -- kamal deploy`.
  - The `.kamal/env.secret` file will reference these environment variables.
- **Dockerfile:** A `Dockerfile` will be created to containerize the Python application, including installing dependencies from `requirements.txt`. The application itself will be run as a long-running process (e.g. the scheduler).
- **Container Registry:** Docker Hub will be used as the container registry. The image will be named `[DOCKERHUB_USERNAME]/ai-thoughts`.
- **Exception Monitoring:** Sentry will be used for tracking exceptions. The `SENTRY_DSN` will be configured via environment variables.
- **LLM Integration:** If enabled, the system will integrate with the OpenAI API using the `openai` library. It will require the `OPENAI_API_KEY` secret.

## 11. Backup Strategy

A shell script will be responsible for creating daily backups.

- **Script Location:** `scripts/backup.sh`
- **Frequency:** The script will be triggered daily by a system cron job (e.g., at 3:00 AM local time).
- **Backup Content:** The script will create a compressed tarball containing:
  - The SQLite database (`data/messages.db`)
  - The Kamal deployment configuration (`config/deploy.yml`)
- **Process:**
  1.  Create a temporary tarball.
  2.  Compress it using `gzip`.
  3.  Create a final filename with a timestamp (`%Y%m%d%H%M%S-ai-thoughts.tar.gz`).
  4.  Move the temporary file to its final destination at `/backups/ai-thoughts/[filename]` to ensure atomicity.
- **Recovery:** The `syncthing` service is expected to run on the server and automatically sync the contents of `/backups/ai-thoughts` to an external location (e.g., a NAS). Manual recovery would involve retrieving a backup from the external location and restoring the files.
