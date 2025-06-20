## Relevant Files

- `tests/test_config.py`
- `src/ai_thoughts/config.py`
- `.env.example`
- `tests/test_models.py`
- `src/ai_thoughts/models.py`
- `src/ai_thoughts/database.py`
- `tests/test_ingest.py`
- `src/ai_thoughts/ingest.py`
- `tests/test_digest.py`
- `src/ai_thoughts/digest.py`
- `templates/digest.md.j2`
- `templates/digest.html.j2`
- `tests/test_publish.py`
- `src/ai_thoughts/publish.py`
- `tests/test_scheduler.py`
- `src/ai_thoughts/main.py`
- `Dockerfile`
- `config/deploy.yml`
- `.kamal/env.secret`
- `scripts/backup.sh`

### Notes

- **Follow TDD**: Tests in the `tests/` directory must be written before implementation code in `src/ai_thoughts/`.
- **Safe Iteration**: Generate one failing test at a time, then write the minimum code to make it pass.
- **Testing Commands**: Use `pytest tests/path/to/specific_test.py` to run individual test files.
- **External Services**: Use `pytest-httpx` to mock external HTTP APIs (Telegram, Substack) and test both success and failure cases.
- **Dependencies**: Add new dependencies to `pyproject.toml` using `uv pip install [package]`.
- **Configuration**: Use `.env` for local development, loading into the Pydantic `Settings` model. Production secrets are injected by `1Password` via `op run -- kamal ...`.
- **Database**: Use an in-memory SQLite database for fast, isolated tests.
- **AI Agent Guidance**: Always write the failing test first, run it to confirm the error, write minimal code to pass, and then refactor. When in doubt, ask for clarification.

## Tasks

- [ ] 1. **Project & Environment Setup**

  - [ ] 1.1. Create the `src/ai_thoughts` and `tests/` directory structure.
  - [ ] 1.2. Use `uv pip install "pydantic[dotenv]"` to add configuration libraries.
  - [ ] 1.3. In `tests/test_config.py`, write a test to assert that a `Settings` class can be initialized and loads a dummy variable from the environment.
  - [ ] 1.4. In `src/ai_thoughts/config.py`, implement the `Settings` class using `pydantic_settings.BaseSettings` to make the test pass.
  - [ ] 1.5. Create the `.env.example` file, listing all required environment variables (`TELEGRAM_API_ID`, `DATABASE_URL`, `SENTRY_DSN`, etc.) with placeholder values.

- [ ] 2. **Core Data Models & Schema**

  - [ ] 2.1. Use `uv pip install sqlmodel` to add the ORM library.
  - [ ] 2.2. In `tests/test_models.py`, write a test that creates an instance of `Message` and `Digest` models and saves them to an in-memory SQLite database.
  - [ ] 2.3. In `src/ai_thoughts/models.py`, define the `Message` and `Digest` `SQLModel` classes to make the test pass. Include fields as specified in the PRD.
  - [ ] 2.4. In `src/ai_thoughts/database.py`, create a function `create_db_and_tables` and another to get a `Session`.
  - [ ] 2.5. Update the test to use the `database.py` functions.

- [ ] 3. **Telegram Ingestion Service**

  - [ ] 3.1. Use `uv pip install telethon`.
  - [ ] 3.2. In `tests/test_ingest.py`, write a contract test for an `IngestService` class, asserting it responds to `connect` and `fetch_history`.
  - [ ] 3.3. In `src/ai_thoughts/ingest.py`, create the skeleton `IngestService` to pass the contract test.
  - [ ] 3.4. In `tests/test_ingest.py`, write an integration test using a mocked `TelethonClient` that verifies `fetch_history` calls the client and saves the resulting messages to a mock database session.
  - [ ] 3.5. Implement the real `fetch_history` logic in `IngestService`.
  - [ ] 3.6. Create a runnable script block (`if __name__ == "__main__"`) in `ingest.py` to execute the service.

- [ ] 4. **Digest Generation Service**

  - [ ] 4.1. Use `uv pip install jinja2 markdown-it-py`.
  - [ ] 4.2. In `tests/test_digest.py`, write a test for a `DigestService` that accepts a list of `Message` objects and verifies it returns a formatted Markdown string.
  - [ ] 4.3. In `src/ai_thoughts/digest.py`, create the skeleton `DigestService`.
  - [ ] 4.4. Create `templates/digest.md.j2` and `templates/digest.html.j2`.
  - [ ] 4.5. Implement the `DigestService` logic to load templates and render the digest.
  - [ ] 4.6. Add a test to verify the service can query the database for messages from the last 7 days.
  - [ ] 4.7. Implement the database query logic.

- [ ] 5. **Substack Publishing Service**

  - [ ] 5.1. Use `uv pip install httpx pytest-httpx`.
  - [ ] 5.2. In `tests/test_publish.py`, write a contract test for a `PublishService` asserting it responds to `publish_draft`.
  - [ ] 5.3. In `src/ai_thoughts/publish.py`, create the skeleton `PublishService`.
  - [ ] 5.4. In `tests/test_publish.py`, write an integration test using `pytest-httpx` to mock the Substack GraphQL API. Verify a `POST` request is sent with the correct HTML payload and auth headers.
  - [ ] 5.5. Implement the `publish_draft` logic using `httpx` to send the request.
  - [ ] 5.6. Add a test for handling a `4xx` or `5xx` error from the Substack API.
  - [ ] 5.7. Implement the error handling.

- [ ] 6. **Scheduling & Main Application Loop**

  - [ ] 6.1. Use `uv pip install apscheduler sentry-sdk openai`.
  - [ ] 6.2. In `src/ai_thoughts/main.py`, create a `main()` function that initializes the Sentry SDK (if `SENTRY_DSN` is set).
  - [ ] 6.3. In `tests/test_scheduler.py`, write a test to verify that a main scheduling function configures and runs a job weekly. Use mocks for the actual services (`DigestService`, `PublishService`).
  - [ ] 6.4. In `main.py`, implement the APScheduler setup, wiring it to call the digest and publish services.

- [ ] 7. **Containerization & Deployment Configuration**

  - [ ] 7.1. Create a multi-stage `Dockerfile` in the project root. The final stage should be a slim production image that runs `python -m ai_thoughts.main`.
  - [ ] 7.2. Create `config/deploy.yml` with the service name (`ai-thoughts`), image name (`[DOCKERHUB_USERNAME]/ai-thoughts`), and server details from the PRD.
  - [ ] 7.3. Create `.kamal/env.secret` and list all the required production secrets (e.g., `TELEGRAM_API_ID`, `SENTRY_DSN`, etc.) that Kamal will source from the environment.

- [ ] 8. **Backup Implementation**
  - [ ] 8.1. Create the `scripts/` directory.
  - [ ] 8.2. In `scripts/backup.sh`, implement the backup logic: create a gzipped tarball of the SQLite DB and `config/deploy.yml` with a timestamp, and move it atomically to `/backups/ai-thoughts/`.
  - [ ] 8.3. Add a section to `README.md` explaining how to set up a system cron job to run the backup script daily.
