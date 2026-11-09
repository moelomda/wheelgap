# Recipes

Practical ways to check your project's dependencies with wheelgap.

## Free-threading readiness

List packages that need a source build or have no usable distribution on at
least one common native platform for free-threaded Python 3.14. With `wheelgap`
and `jq` installed, run this in your project directory:

```bash
set -o pipefail
wheelgap -p 3.14t -t all --json --exit-zero \
  | jq -r '.packages[] | select([.results[]] | any(. == "source" or . == "missing")) | .name'
```

- `-p 3.14t` checks free-threaded Python; you do not need to run wheelgap on that
  interpreter.
- `-t all` checks the eight common native platforms. Replace it with the targets
  you deploy to if you only need a subset.
- `--exit-zero` keeps known wheel gaps from failing this informational command.
  Bash's `pipefail` still exposes input errors from wheelgap.
- The filter reads the **values** of `results`. It also works with several
  Python versions (for example, `-p 3.13,3.14t`), when the result keys include
  both the Python version and platform.

For example, running in this repository's `tests/fixtures/uvproj` directory
prints:

```text
fastlib
nativeext
puresdist
winonly
```

This reports wheel availability, not whether a package is safe to use without
the GIL. An empty list means no `source` or `missing` results; inspect any lookup
errors in stderr or the JSON's package `error` fields before treating it as a
clean result.
