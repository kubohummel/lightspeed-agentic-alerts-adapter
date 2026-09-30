# Contributing

## Getting started

1. Fork and clone the repository
2. Install prerequisites: Go 1.26+, [golangci-lint](https://golangci-lint.run/)
3. Run `make test` and `make lint` to verify your setup

## Development workflow

### Proposing changes

Start with the [spec index](.ai/spec/README.md). Behavioral contracts live in `.ai/spec/what/`; implementation guides live in `.ai/spec/how/`.

1. Describe the change in the affected spec and mark new behavior `[PLANNED]`.
2. Get the proposal reviewed and approved.
3. Implement the change and check it against the spec.
4. Update implementation guides and replace planned markers when the behavior is implemented.
5. Update the parent Lightspeed specs when a contract spans repositories.

Keep existing rule identifiers stable. Use sub-numbers for new rules and record important design decisions in `.ai/spec/decisions/`, or in the parent workspace for shared decisions.

### Code changes

1. Create a feature branch from `main`
2. Make your changes
3. Ensure all checks pass:
   ```sh
   make fmt
   make vet
   make lint
   make test
   ```
4. Open a pull request

### Commit messages

Follow [Conventional Commits](https://www.conventionalcommits.org/):

```
feat: add alertmanager retry logic
fix: guard against nil alert fingerprint
test: cover AlreadyExists path
docs: update architecture diagram
refactor: return (bool, error) from CreateAgenticRun
```

## Code review

Pull requests require approval from a reviewer listed in the [OWNERS](OWNERS) file.

## License

By contributing, you agree that your contributions will be licensed under the [Apache License 2.0](LICENSE).
