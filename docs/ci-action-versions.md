# CI GitHub Action Versions

The following versions are canonical for the CI jobs in this project. When updating any workflow, ensure each job uses the required version so all task docs stay consistent.

| Job | Action | Required Reference |
| --- | --- | --- |
| lint | actions/checkout | actions/checkout@v4 |
| lint | actions/setup-go | actions/setup-go@v5 |
| test | actions/checkout | actions/checkout@v5 |
| test | actions/setup-go | actions/setup-go@v6 |

If a version needs to change, update this manifest first and then adjust all referenced workflow files and task docs to match.
