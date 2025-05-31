# action-dedupe

An action that wraps [fkirc/skip-duplicate-actions](https://github.com/fkirc/skip-duplicate-actions) with common configuration to identify duplicate workflow runs.

This repository is not meant to be referenced in third-party workflows; please fork the repository if you would like to use it in your project.

## Inputs

| Name         | Description                                                                                                                              | Required | Default |
| ------------ | ---------------------------------------------------------------------------------------------------------------------------------------- | :------: | ------- |
| github_token | The Github authentication token.                                                                                                         |    ✅    |         |
| ignore       | Whether to ignore the determination. Used to conditionally force something to run even if it's skippable.                                |          | "false" |
| paths        | A YAML sequence of paths to include in the determination.                                                                                |          | ""      |
| paths_ignore | A YAML sequence of paths to exclude from the determination                                                                               |          | ""      |
| paths_named  | A YAML mapping with named `paths`/`paths_ignore` conditions. Used to determine skippability for e.g., multiple jobs/steps in a workflow. |          | ""      |

## Outputs

| Name        | Description                                         |
| ----------- | --------------------------------------------------- |
| should_skip | Whether the run should be skipped.                  |
| named       | Whether each named condition run should be skipped. |

## Examples

```
jobs:
  changes:
    runs-on: ubuntu-latest
    outputs:
      should_skip: ${{ steps.dedupe.outputs.should_skip }}
      named: ${{ steps.dedupe.outputs.named }}
    steps:
      - uses: lojoja/action-dedupe@main
        id: dedupe
        with:
          github_token: ${{ github.token }}
          paths: |
            - "**.ya?ml"
          paths_ignore: |
            - .gitignore
          paths_named: |
            foo:
              paths:
                - "**.json"

  unnamed-change:
    runs-on: ubuntu-latest
    needs: changes
    if: needs.changes.outputs.should_skip == 'false'
    ...

  named-change:
    runs-on: ubuntu-latest
    needs: changes
    if: ! fromJSON(needs.changes.outputs.named).foo.should_skip
    ...
```

## License

action-dedupe is released under the [MIT License](./LICENSE)
