<h1 align="center"> PyLibUpdater </h1> <br>

<p align="center" style="font-weight:bold">
    Github Action for managing your Python libraries across multiple repositories
</p>


Introducing the "Python Library Updater" Github Action, the ultimate tool for managing your Python libraries across multiple repositories. This action is specifically designed to streamline the process of updating your Python libraries to their latest version, making it easy for you to maintain consistency across all your repositories.


## Table of Contents

- [Table of Contents](#table-of-contents)
- [Parameters](#parameters)
- [Requirements](#requirements)
- [Usage](#usage)
- [Release procedure](#release-procedure)

## Parameters

| Parameter       | Description                                                | Required |
|-----------------|------------------------------------------------------------|----------|
| library         | The name of the library                                    | true     |
| new_version     | The new version to update to                               | true     |
| repository      | The repository full name where to create the PR            | true     |
| branch          | The branch where to create the PR to                        | true     |
| python_version  | The python version used on the repository                  | true     |
| pat             | Github Personal token REPO scope                           | true     |
| ssh_private_key | SSH key to pull libraries                                  | false    |
| ssh_library_url | SSH url from the library to pull                           | false    |




## Requirements

1. PAT: You need to create a PAT (Personal Access Token) with the following permissions:
    - repo: Full control of private repositories
    - workflow: Read and write access to actions, workflows, and related artifacts.
2. ( Optionally ) SSH_PRIVATE_KEY:  If you want to use SSH to install the libraries, you need to create a SSH key and add it to your Github Account or repository secrets.

## Usage

This action is designed to be used in a workflow triggered by a tag push. The tag name will be used as the new version of the library. The action will then update the version of the library in the `pyproject.toml` file and create a PR with the changes. 

```yaml
name: On push version

on:
  push:
    tags: # If you only want to trigger the worklfow only with the minor tags (v1.0, v1.1, v1.2) and not majors (v0, v1, v2) you can use the following regex
      - 'v[0-9]+.[0-9]+'

jobs:
  format:
    name: Update test
    runs-on: ubuntu-latest
    strategy:
      matrix:
        repositories:
          - name: 'joanplaja/django-poetry-example'
            python_version: '3.8'
            branch: 'master'

    steps:
      # The tag abstraction logic it's outside the action, because it's not related to the action itself.
      - name: Get tag
        id: set-tag
        run: |
          REF="${{ github.event.ref }}"
          echo "Original ref: $REF"

          # Remove prefix
          TAG="${REF/refs\/tags\//}"
          echo "Modified ref: $TAG"

          # Set modified ref as an output
          echo "::set-output name=tag::$TAG"

      - name: 🏗 Update python library and create PR
        id: pylibupdater
        uses: joanplaja/pylibupdater@v0
        with:
          library: 'python_library_example'
          new_version: ${{ steps.set-tag.outputs.tag }}
          repository: ${{ matrix.repositories.name }}
          branch: ${{ matrix.repositories.branch }}
          python_version: ${{ matrix.repositories.python_version }}
          pat: ${{ secrets.PAT }}
          ssh_private_key: ${{ secrets.SSH_PRIVATE_KEY }}
          ssh_library_url: git+ssh://git@github.com/joanplaja/python_library_example@${{ steps.set-tag.outputs.tag }}
```

## Release procedure

After a new commit, we need to create a new tag. To do so first get the last tag:
```
git fetch
git describe --tags --match "*.*" `git rev-list --tags --max-count=1`
```
Create a new one with bumping the last number: ( for example if last tag is v1.9)
```
git tag v1.10 -m "Bump version"
```
If this version is stable, make the major tag point to the last commit:
```
git tag -fa v1 -m "Update major tag"
```
Push the new commit:
```
git push origin main
```
Push the new tags:
```
# We need to force push v1 because it may exists if it's not the first major push.
git push origin v1.10
git push -f origin v1
```