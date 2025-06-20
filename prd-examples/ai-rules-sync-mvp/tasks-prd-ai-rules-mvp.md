## Relevant Files

- `go.mod` - Go module definition and dependencies.
- `cmd/ai-rule-sync/main.go` - Main application entry point.
- `cmd/ai-rule-sync/commands.go` - Cobra command definitions.
- `internal/config/config.go` - Configuration loading and saving logic.
- `internal/config/config_test.go` - Unit tests for configuration (TDD - create first).
- `internal/repo/repo.go` - Git repository management logic (clone, pull).
- `internal/repo/repo_test.go` - Unit tests for repository logic (TDD - create first).
- `internal/sync/init.go` - Implementation of the `init` command logic.
- `internal/sync/init_test.go` - Unit tests for init command (TDD - create first).
- `internal/sync/pull.go` - Implementation of the `pull` command logic with drift detection.
- `internal/sync/pull_test.go` - Unit tests for pull command (TDD - create first).
- `internal/sync/publish.go` - Implementation of the `publish` command logic.
- `internal/sync/publish_test.go` - Unit tests for publish command (TDD - create first).
- `.goreleaser.yml` - GoReleaser configuration for builds and releases.
- `scripts/setup-test-env.sh` - Script to create a test environment for manual verification.
- `scripts/test-init.sh` - Script to test the init command with automated input.
- `scripts/test-init-private.sh` - Script to test init with private SSH repositories.
- `scripts/test-init-all.sh` - Comprehensive test script for both HTTPS and SSH repositories.
- `scripts/test-pull.sh` - Script to test the pull command functionality.
- `scripts/test-pull-private.sh` - Script to test pull with private repository.
- `scripts/test-publish.sh` - Script to test the publish command functionality.
- `scripts/test-publish-private.sh` - Script to test publish with private repository.
- `scripts/test-cd.sh` - Script to test the cd command functionality.
- `scripts/test-cd-private.sh` - Script to test cd with private repository.
- `.gitignore` - Git ignore file including test directories and temporary files.
- `Makefile` - Build and development tasks.

### Notes

- **Follow TDD:** Tests should be written before implementation code. For a CLI, this means writing tests for functions in the `internal` packages that will be called by the commands.
- **Safe Iteration:** Focus on one parent task at a time, implementing sub-tasks incrementally.
- **Testing:** Use `go test ./...` to run all tests. To run tests for a specific package, `cd` into the directory and run `go test`.
- **Dependencies:** Use `go get` to add new dependencies and `go mod tidy` to clean up `go.mod`.
- **AI Agent Guidance:** Always write the failing test first, then write the minimal code to make it pass. Commit the code only when all tests for the feature pass.
- **Test Repository:** Unit tests use public repositories (e.g., `https://github.com/octocat/Hello-World.git`) for reliability. Manual verification uses the user's actual ai-rules repository (`https://github.com/kovyrin/ai-rules.git`).
- **Test Environment:** Run `./scripts/setup-test-env.sh` to create a test environment in `test-repos/` for manual verification of CLI commands.
- **Git Ignore:** Test repositories and temporary files are excluded via `.gitignore` to keep the repository clean.
- **SSH Support:** The repo package automatically detects SSH URLs (git@ or ssh://) and configures authentication using common SSH key locations (~/.ssh/id_ed25519, id_rsa, etc.).

## Tasks

- [x] 1. **Project Scaffolding & CLI Structure**

  - [x] 1.1. Initialize the Go module: `go mod init github.com/your-name/ai-rule-sync`.
  - [x] 1.2. Add Cobra as a dependency: `go get github.com/spf13/cobra@latest`.
  - [x] 1.3. Verify the basic CLI skeleton runs by executing `go run ./cmd/ai-rule-sync`.
  - [x] 1.4. Verify by running `make help` and `make run`.

- [x] 2. **Configuration Management**

  - [x] 2.1. Create `internal/config/config.go` and `internal/config/config_test.go`.
  - [x] 2.2. In `config_test.go`, write a test for `Load()` that checks for a non-existent config file and asserts that a default, empty config struct is returned without an error.
  - [x] 2.3. Implement the `Load()` function in `config.go` to make the test pass. Use `os.UserHomeDir()` to find the home directory and construct the path `~/.ai-rule-sync/config.yaml`.
  - [x] 2.4. In `config_test.go`, write a test for `Save()` that saves a sample config struct to a temporary file and then verifies the file's contents are correct YAML.
  - [x] 2.5. Implement the `Save()` function in `config.go`. It should use a YAML library (like `gopkg.in/yaml.v3`) to marshal the struct and write it to the file.
  - [x] 2.6. In `config_test.go`, add a test case to `Load()` that reads the temporary file you created in the `Save()` test and asserts the struct is populated correctly.
  - [x] 2.7. Verify by running `make test`.

- [x] 3. **Core Repository Logic**

  - [x] 3.1. Add the `go-git` dependency: `go get github.com/go-git/go-git/v5`.
  - [x] 3.2. Create `internal/repo/repo.go` and `internal/repo/repo_test.go`.
  - [x] 3.3. In `repo_test.go`, write a test for `Clone()` that attempts to clone a public repository (e.g., `https://github.com/octocat/Hello-World.git`) into a temporary directory and asserts that no error occurs and the directory is not empty.
  - [x] 3.4. Implement the `Clone()` function in `repo.go` using `git.PlainClone`.
  - [x] 3.5. In `repo_test.go`, write a test for `Pull()` that uses the repo cloned in the previous step and asserts that a pull operation completes without errors.
  - [x] 3.6. Implement the `Pull()` function in `repo.go`.
  - [x] 3.7. Verify by running `make test`.
  - [x] 3.8. Add SSH authentication support for private repositories (detects SSH URLs and uses SSH keys).

- [x] 4. **Implement `init` Command**

  - [x] 4.1. In `commands.go`, refactor the `initCmd.Run` logic into a new function `RunInit` in `internal/sync/init.go`.
  - [x] 4.2. In `internal/sync/init_test.go`, write an integration test for `RunInit`. This test will need to mock user input for the Git URL. It should assert that after `RunInit` is called, a config file exists and a repository has been "cloned" (you can mock the `repo.Clone` function).
  - [x] 4.3. Implement `RunInit` to orchestrate the logic: call `config.Load`, prompt for URL if needed, call `repo.Clone`, and `config.Save`.
  - [x] 4.4. Wire the `initCmd` to call `sync.RunInit`.
  - [x] 4.5. Verify by running `./scripts/test-init-all.sh` to test both HTTPS and SSH repositories, including the private `git@github.com:kovyrin/ai-rules.git`.

- [x] 5. **Implement `pull` Command**

  - [x] 5.1. In `commands.go`, refactor the `pullCmd.Run` logic into `RunPull` in `internal/sync/pull.go`.
  - [x] 5.2. In `internal/sync/pull_test.go`, write a test for drift detection. Set up a temporary project directory and a temporary local repo with a file. Modify the file in the project directory. Assert that `RunPull` returns a specific "drift detected" error.
  - [x] 5.3. Implement the drift detection logic in `RunPull` by comparing file contents.
  - [x] 5.4. In `pull_test.go`, write a test for the success path (no drift). Assert that `repo.Pull` is called (using a mock) and that files are copied correctly.
  - [x] 5.5. Implement the success path logic in `RunPull`.
  - [x] 5.6. Wire the `pullCmd` to call `sync.RunPull`.
  - [x] 5.7. Verify by running `make build && ./ai-rule-sync pull`.

- [x] 6. **Implement `publish` Command**

  - [x] 6.1. In `commands.go`, refactor the `publishCmd.Run` logic into `RunPublish` in `internal/sync/publish.go`.
  - [x] 6.2. In `internal/sync/publish_test.go`, write a test that initializes a temporary Git repository. The test should call `RunPublish` with a new file path.
  - [x] 6.3. Assert that the file is copied into the repo and that a new commit has been created in the test repository with the correct commit message.
  - [x] 6.4. Implement `RunPublish` to copy the file and use `go-git` to add and commit it. (Note: Pushing will be handled manually by the user after the commit is made).
  - [x] 6.5. Wire the `publishCmd` to call `sync.RunPublish`.
  - [x] 6.6. Verify by running `make build && ./ai-rule-sync publish`.

- [x] 7. **Implement `cd` Command**

  - [x] 7.1. In `config_test.go`, write a test for a new function `GetRepoPath()` that returns the correct, absolute path to the local rules repo.
  - [x] 7.2. Implement `GetRepoPath()` in `config.go`.
  - [x] 7.3. Update the `cdCmd.Run` function to call `config.GetRepoPath()` and print the result to standard output.
  - [x] 7.4. Verify by running `make build && ./ai-rule-sync cd`.

### Verification Notes

- **Private Repository Access:** Successfully tested `init` command with the user's private `git@github.com:kovyrin/ai-rules.git` repository using SSH authentication.
- **SSH Key Support:** The implementation automatically searches for SSH keys in common locations and configures authentication for go-git.
- **Test Scripts:** Created comprehensive test scripts (`test-init-private.sh` and `test-init-all.sh`) for verifying both HTTPS and SSH repository access.
- **Pull Command:** Successfully implemented with drift detection that prevents overwriting local changes. Tested with both public and private repositories.
- **Drift Detection:** The pull command correctly detects both modified files and new local files, preventing accidental data loss.
- **Publish Command:** Successfully implemented with automatic commit creation. Detects whether a file is new (Add) or existing (Update) for appropriate commit messages.
- **Git Integration:** The publish command uses go-git to create proper commits with user information from git config when available.
- **CD Command:** Simple but effective command that outputs the repository path for easy navigation. Works with shell command substitution: `cd $(ai-rule-sync cd)`.

- [ ] 8. **Build & Release Automation**
  - [ ] 8.1. Create a `.goreleaser.yml` file in the project root.
  - [ ] 8.2. Configure the `builds` section to target macOS, Linux, and Windows for amd64 and arm64 architectures.
  - [ ] 8.3. Configure the `archives` section to create `.tar.gz` and `.zip` packages.
  - [ ] 8.4. Add a `release` section to automatically publish to GitHub Releases when a new tag is pushed.
  - [ ] 8.5. Create a GitHub Actions workflow file at `.github/workflows/release.yml` that triggers on `push: {tags: ['v*']}` and runs the `goreleaser/goreleaser-action`.
  - [ ] 8.6. Verify by checking the GitHub releases page for a new release.
