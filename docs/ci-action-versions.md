# CI GitHub Action Versions

The following versions are canonical for the CI jobs in this project. When updating any workflow, ensure each job uses the required version so all task docs stay consistent.

| Job | Action | Required Reference |
| --- | --- | --- |
| lint | actions/checkout | actions/checkout@v5 |
| lint | actions/setup-go | actions/setup-go@v6 |
| test | actions/checkout | actions/checkout@v5 |
| test | actions/setup-go | actions/setup-go@v6 |

If a version needs to change, update this manifest first and then adjust all referenced workflow files and task docs to match.

## Automation

- Add a repository check (`make verify-action-versions`) that parses every `.github/workflows/*.yml` file, extracts `uses: owner/repo@ref` entries, and verifies each action reference appears in the manifest table above. The check must fail the build if any workflow action is missing from this manifest or if the manifest lists actions not present in workflows.
- Wire `make verify-action-versions` into the CI pipeline (e.g., a dedicated job in `ci.yml`) so pull requests cannot merge unless workflow changes keep this manifest accurate.
