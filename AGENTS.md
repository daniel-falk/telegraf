# AGENTS.md

Telegraf is InfluxData's plugin-driven agent for collecting, processing,
aggregating, and writing metrics. It is a single Go module that compiles to one
standalone static binary (`./cmd/telegraf`). There is no database or web service
to run; you configure it with a TOML file listing input/processor/output plugins
and run the binary.

Standard developer commands are documented in `CONTRIBUTING.md` and the
`Makefile` (`make help`). Common ones: `make build`, `make test`, `make lint`,
`make config`, `make docs`.

## Cursor Cloud specific instructions

- Go toolchain: the system `go` is 1.22.2, but `go.mod` sets `go 1.26.0`, so Go's
  automatic toolchain feature downloads and uses `go1.26.0` transparently. Just
  run `go`/`make` normally; do not pin `GOTOOLCHAIN=local` (that forces 1.22.2
  and fails). The first `make build` is slow (~4 min) because it downloads the
  large dependency set and compiles ~245 input plugins; subsequent builds are
  cached.

- Build/run: `make build` produces a `./telegraf` binary (~400 MB). Run it with
  a config, e.g. create a TOML with `[[inputs.cpu]]`, `[[inputs.mem]]`,
  `[[outputs.file]]` and run `./telegraf --config <file>`. Use
  `make config` to regenerate `etc/telegraf.conf` from the current plugin set.

- Tests: `make test` runs `go test -short -race ./...` across the whole tree and
  is very slow. When iterating, test only the affected packages, e.g.
  `go test -short ./plugins/inputs/cpu/... ./config/...`. Integration tests
  (`make test-integration`) require external services (Docker containers) and are
  not needed for most changes.

- Linters (`make lint`) require `golangci-lint` and `markdownlint`, which are
  pre-installed in the snapshot on `PATH` (`~/go/bin` and `~/.npm-global/bin`,
  wired up in `~/.bashrc`). IMPORTANT gotcha: `golangci-lint` must be built with
  the same Go version the project targets. A `go install golangci-lint@...` picks
  up golangci-lint's own toolchain (go1.25) and then refuses to lint with
  "Go language version used to build golangci-lint is lower than the targeted Go
  version". If you ever need to reinstall it, force the toolchain:
  `GOTOOLCHAIN=go1.26.0 go install github.com/golangci/golangci-lint/v2/cmd/golangci-lint@v2.11.4`.
  `golangci-lint run` with no path args lints the entire repo and is slow; scope
  it to changed packages (e.g. `golangci-lint run ./plugins/inputs/cpu/...`).

- `npm install -g` fails with EACCES against the default global prefix. Do not
  set a global npm `prefix` in `~/.npmrc` — it conflicts with `nvm`. `markdownlint`
  is already installed under `~/.npm-global`; that is sufficient for `make lint`.
