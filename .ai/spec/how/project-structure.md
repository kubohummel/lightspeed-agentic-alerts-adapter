# Project Structure

## Module Map

| File/Directory | Key Symbols | Responsibility |
|---|---|---|
| `cmd/alerts-adapter/main.go` | `main`, `newTargets`, `newTargetRegistry`, `startSpokeClusterController` | Entry point: wires dependencies, handles signals, starts poll loop and optional SpokeCluster controller |
| `internal/adapter/adapter.go` | `Adapter`, `Run`, `reconcile`, `reconcileTarget`, `AlertSource`, `AgenticRunClient`, `SuspensionSource`, `Target`, `TargetSource` | Poll loop and stateless deduplication. Defines consumer interfaces. Implements receiver filtering, pre-run delay, active-run check, post-run delay, creation backoff |
| `internal/agenticrun/build.go` | `Build`, `BuildForTarget`, `StableFingerprint`, `NextAvailableName`, `requestData`, `requestTemplate` | Alert-to-AgenticRun translation: deterministic naming, metadata sanitization, FNV-64a fingerprinting, Go template rendering |
| `internal/agenticrun/client.go` | `Client`, `ListAgenticRuns`, `CreateAgenticRun`, `SpokeTargetID` | Kubernetes CRUD for AgenticRun CRs. Target-scoped listing and creation. Spoke target ID computation |
| `internal/agenticrun/request.tmpl` | (template) | Embedded Go template for the `spec.request` field |
| `internal/alertmanager/client.go` | `Client`, `GetAlerts`, `Config` | AlertManager v2 API client. Bearer token auth, TLS via service CA or custom CA bundle. Supports local and remote endpoints |
| `internal/config/config.go` | `Config`, `LoadFromFile`, `Default`, `ToolsConfig`, `AgentConfig` | Configuration read from a mounted YAML file at startup. YAML parsing, defaults, duration clamping, ignored labels, skills validation |
| `internal/agenticolsconfig/client.go` | `Client`, `Suspended` | Reads `AgenticOLSConfig` singleton for suspended mode. Treats missing CRD/object as not suspended |
| `internal/multicluster/controller.go` | `Reconciler`, `Reconcile`, `SetupWithManager` | controller-runtime reconciler for SpokeCluster watch. Adds, replaces, or removes targets on lifecycle events |
| `internal/multicluster/registry.go` | `Registry`, `SetSpoke`, `RemoveSpoke`, `Targets` | Thread-safe target registry combining fixed local targets with dynamic spoke targets |
| `internal/multicluster/target.go` | `BuildTarget`, `CredentialSecretLabel` | Constructs a reconciliation target from a SpokeCluster and its credential Secret |

## Key Entry Points

- `cmd/alerts-adapter/main.go:main()` parses `--multicluster`, reads startup configuration, creates the Kubernetes client, builds initial targets, optionally starts the SpokeCluster controller, and calls `adapter.Run()`.
- `adapter.Run()` executes an immediate reconcile then enters a ticker loop until context cancellation.

## Naming Conventions

- One package per concern, named for the domain concept (not the implementation pattern).
- Interfaces defined in the consumer package (`adapter`), not the provider.
- Test files colocated with source: `*_test.go` in the same package.
- E2E tests in `test/e2e/`.

## Dependencies and Logging

The six internal packages are wired together in `cmd/alerts-adapter/main.go`. AgenticRun and suspension types come from `github.com/openshift/lightspeed-agentic-operator/api`; SpokeCluster types come from `github.com/openshift/lightspeed-hub/api`. Alert retrieval uses `github.com/prometheus/alertmanager` and its generated v2 client. Kubernetes clients use controller-runtime; creation backoff uses client-go's `flowcontrol.Backoff`.

Logging uses `log/slog` with a JSON handler at Info level. Main passes the logger to components and sets the default logger. Controller-runtime logging uses an adapter over the same handler.

See [reconcile flow](reconcile-flow.md) for call order, state, and known implementation limits.
