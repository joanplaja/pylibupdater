<h1 align="center"> PyLibUpdater </h1> <br>

<p align="center" style="font-weight:bold">
    Github Action for managing your Python libraries across multiple repositories
</p>


Introducing the "Python Library Updater" Github Action, the ultimate tool for managing your Python libraries across multiple repositories. This action is specifically designed to streamline the process of updating your Python libraries to their latest version, making it easy for you to maintain consistency across all your repositories.

To use this action, you need to have the "poetry" package manager installed in your repository. With "Python Library Updater" you can automatically check for updates to your Python libraries and push those updates to all your repositories, saving you valuable time and effort. Whether you're managing a large or small set of repositories, this action provides a simple and efficient way to ensure that your Python libraries are always up to date.

This action is easy to integrate into your existing workflows, and can be customized to fit your specific needs. Say goodbye to the hassle of manual updates and embrace the power of "Python Library Updater" for your next library.


## Table of Contents

- [Table of Contents](#table-of-contents)
- [Usage](#usage)
- [Release procedure](#release-procedure)

## Usage

```
- name: 🏗 Update python library and create PR
  id: pylibupdater
  uses: joanplaja/pylibupdater
  with:
    library: 'black'
    new_version: '23.3.0'
    repository: 'joanplaja/django-poetry-example'
    branch: 'master'
    python_version: '3.8'
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