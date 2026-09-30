# Lightspeed Agentic Alerts Adapter

A Go component that polls OpenShift AlertManager and creates `AgenticRun` resources for alerts that pass filtering and deduplication. It runs as one replica and only creates runs. It has no persistent storage; target discovery and creation backoff use memory.

## Specs

All specifications live in `.ai/spec/`. Start with [.ai/spec/README.md](.ai/spec/README.md) for the project overview, reading order, and structure guide.

- Behavioral contracts: `.ai/spec/what/`.
- Code map and reconcile flow: `.ai/spec/how/`.
- Human architecture overview: [ARCHITECTURE.md](ARCHITECTURE.md).
- Cross-repo contracts: the parent workspace's `.ai/spec/`.
- `CLAUDE.md` is a symlink to this file. Edit instructions here.

## Commands

```sh
make build           # Build ./bin/alerts-adapter
make test            # Run Go tests, excluding test/e2e
make lint            # Run golangci-lint
make fmt             # Run go fmt
make vet             # Run go vet
make coverage        # Create coverage.html; includes all Go packages
make container-build # Build the image with podman
make container-push  # Build and push; set IMAGE_NAME and IMAGE_TAG
make test-e2e        # Run E2E tests against a prepared cluster

# Run a single test or subtest
go test -run TestFunctionName ./internal/adapter/
go test -run TestFunctionName/subtest_name ./internal/adapter/
```

## Conventions

- Use clear English for replies, comments, and documentation.
- Commit messages use Conventional Commits (`feat:`, `fix:`, `test:`, `docs:`, `refactor:`).
- Use structured JSON logging with `log/slog`. Pass loggers explicitly; `slog.SetDefault` is set in main.
- Define interfaces in the consumer package (`adapter`).
- Use table-driven tests.
- Keep spec rule identifiers stable. Mark unimplemented requirements with `[PLANNED]` and explain current limits.
