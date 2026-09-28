# gendiff

[![Actions Status](https://github.com/pavloveone/gendiff-go/actions/workflows/ci.yml/badge.svg)](https://github.com/pavloveone/gendiff-go/actions)
[![Coverage](https://sonarcloud.io/api/project_badges/measure?project=pavloveone_gendiff-go&metric=coverage)](https://sonarcloud.io/summary/new_code?id=pavloveone_gendiff-go)
[![Quality Gate Status](https://sonarcloud.io/api/project_badges/measure?project=pavloveone_gendiff-go&metric=alert_status)](https://sonarcloud.io/summary/new_code?id=pavloveone_gendiff-go)

**Demo:** [asciinema recording](https://asciinema.org/a/QIlzt5NyC1YojONz)

A CLI tool that compares two configuration files (JSON or YAML) and renders a structured diff - including nested objects - in one of three output formats.

Config drift between environments (`dev.json` vs `prod.yaml`) is hard to spot by eye. gendiff parses both files into a common tree, walks it recursively, and prints only what actually changed, in whichever format fits the context: human-readable, script-friendly, or JSON for downstream tooling.

Built as a project for Hexlet's Go developer course. This repo is that project with the module renamed and detached from Hexlet's course infrastructure for standalone use.

## Stack

Go 1.22, urfave/cli/v3, gopkg.in/yaml.v3, testify, golangci-lint, GitHub Actions, SonarCloud

## How it's structured

```
cmd/gendiff          CLI entrypoint (urfave/cli/v3), flag parsing
internal/parsers     public entry point, validates the two-path contract
(root package)       reads files, detects format by extension, unmarshals
                      JSON/YAML into a generic map, builds the diff tree
internal/models      shared types: DiffNode, NodeType, FileData
internal/formatters  Format() dispatches to stylish/plain/json - each
                      formatter only depends on the DiffNode tree, so
                      adding a fourth format doesn't touch the diff logic
```

The diff tree is built by one recursive function that classifies every key as added / removed / changed / unchanged / nested, so arbitrarily deep configs are handled without extra code. Keys are sorted alphabetically at every level, so diffs are stable and diffable themselves. JSON and YAML are normalized to the same internal structure before comparison, so you can diff a JSON file against a YAML file directly.

## Usage

```
gendiff [--format stylish|plain|json] <first file> <second file>
```

```
$ gendiff testdata/fixture/file1.json testdata/fixture/file2.json
{
  - follow: false
    host: hexlet.io
  - proxy: 123.234.53.22
  - timeout: 50
  + timeout: 20
  + verbose: true
}
```

Same comparison with `--format plain`:

```
$ gendiff --format plain testdata/fixture/file1.json testdata/fixture/file2.json
Property 'follow' was removed
Property 'proxy' was removed
Property 'timeout' was updated. From 50 to 20
Property 'verbose' was added with value: true
```

`--format json` gives machine-readable output for piping into other tools.

## Running it locally

```bash
git clone git@github.com:pavloveone/gendiff-go.git
cd gendiff-go
make build          # -> bin/gendiff
./bin/gendiff testdata/fixture/file1.json testdata/fixture/file2.json
```

## Tests

```bash
make test            # go test -v ./...
make test-coverage   # + HTML coverage report
make lint            # golangci-lint
```

Table-driven tests cover the parser, the diff engine, and every formatter, including edge cases: empty files, invalid JSON, mixed JSON/YAML input, wrong argument counts, and deeply nested structures.

## Author

Aleksandr Pavlov
[LinkedIn](https://linkedin.com/in/pavloveone)
