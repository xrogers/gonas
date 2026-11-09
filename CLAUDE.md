# CLAUDE.md

## Builds and tests

Run builds and tests through `scripts/offload`, e.g.
`scripts/offload go test ./internal/...` or `scripts/offload go build -o gonas .`.
It syncs the working tree to the faster desktop PC (ssh host `pc`) and runs the
command there, falling back to a local run when the PC is offline. Its output
says which one happened. Build output (like the `gonas` binary) stays on the
machine that ran the command.

Locally, Go lives in `~/sdk/go/bin`, which may not be on `PATH`.

## Integration tests

The integration packages (`common`, `httpd`, `sftpd`, `ftpd`, `webdavd`) share
fixed ports and a SQLite test database (`gonas.db` at the repo root), so run
them like CI does (`.github/workflows/development.yml`):

- Serially, with the CI build tags:
  `go test -tags nopgxregisterdefaulttypes,disable_grpc_modules -p 1 ./internal/...`
- From a fresh database. `gonas.db` is gitignored, so the offload sync never
  resets it on the PC, and a crashed or parallel run leaves test users behind
  that make later runs fail with 409 Conflict. Before the tests, build `gonas`
  and the `tests/eventsearcher` and `tests/ipfilter` helpers, then run
  `./gonas initprovider` and `./gonas resetprovider --force`. Do the prep and
  the tests in a single `scripts/offload bash -c '...'` call so both run on the
  same machine.
