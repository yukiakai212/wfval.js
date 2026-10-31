# wfval

Generate workflow-friendly string values from paths and URLs.

`wfval` writes one line to standard output. It does not write to
`$GITHUB_OUTPUT` itself, so a workflow remains in control of where the value is
stored.

## Installation

```sh
npm install --save-dev wfval
```

## Commands

```text
wfval url <url> [base-url] [--format <template>] [--name <assignment-name>]
wfval path <operation> [path-arguments...] [--name <assignment-name>]
```

`--name` changes the printed line from `value` to `assignment-name=value`.
It is useful when redirecting output to GitHub Actions' `$GITHUB_OUTPUT` file.

## URL values

The `url` command creates a native `URL` from `<url>` and the optional
`[base-url]`. By default, it prints the serialized URL (`{url}`).

```sh
wfval url 'https://some:400/path?query=value#section'
# https://some:400/path?query=value#section

wfval url 'https://some:400/path' --format '{port}@{hostname}'
# 400@some

wfval url '/path' 'https://some:400' --format '{origin}{pathname}'
# https://some:400/path
```

Use `--format` with any combination of these template fields:

```text
{hash} {hostname} {host} {href} {origin} {password}
{pathname} {port} {protocol} {search} {url}
```

`{url}` uses `URL#toString()`; `{href}` uses `URL#href`.

## Path values

The `path` command supports these `pathe` operations:

| Operation | Arguments |
| --- | --- |
| `basename` | `<path> [suffix]` |
| `dirname` | `<path>` |
| `join` | `[path-segment...]` |
| `normalize` | `<path>` |
| `relative` | `<from-path> <to-path>` |
| `resolve` | `[path-segment...]` |
| `extname` | `<path>` |

All path operations use `pathe`, so results use forward slashes (`/`) across
platforms.

```sh
wfval path join workspace dist assets
# workspace/dist/assets

wfval path relative /workspace /workspace/dist
# dist

wfval path basename /workspace/file.test.ts .ts
# file.test
```

`join` and `resolve` may receive zero or more path segments. Other operations
validate their own required argument count.

## GitHub Actions

Use `--name` and redirect the one-line result yourself:

```yaml
- id: values
  shell: bash
  run: |
    npx wfval path join "$GITHUB_WORKSPACE" dist --name artifact_path >> "$GITHUB_OUTPUT"
    npx wfval url 'https://some:400/path' --format '{port}@{hostname}' --name endpoint >> "$GITHUB_OUTPUT"
```

The step above creates `artifact_path` and `endpoint` outputs without giving
the CLI any direct knowledge of GitHub Actions output files.

## License

[MIT](LICENSE) Yuki Akai
