# Checkout Plus

Checkout Plus is a GitHub Action based on GitHub's [actions/checkout](https://github.com/actions/checkout) action, with extra features added to simplify your build pipelines.  Checkout Plus maintains full compatibility with [actions/checkout](https://github.com/actions/checkout) and can be easily used as a drop-in replacement.

To avoid confusion with differing version numbers, Checkout Plus follows the same versioning as [actions/checkout](https://github.com/actions/checkout) (beginning with v3.3.0), and this fork will integrate any new changes whenever a new release is made there.

Development on Checkout Plus is provided and supported by [Cold Fusion](https://github.com/coldfusionjp), and we actively use it for all of our CI/CD pipelines here on GitHub.

[![Build and Test](https://github.com/coldfusionjp/ghaction-checkout-plus/actions/workflows/test.yml/badge.svg)](https://github.com/coldfusionjp/ghaction-checkout-plus/actions/workflows/test.yml)

# Added Features

- Restore last modified timestamps
  > Useful for incremental builds, enabling `restore-mtime` will ensure the timestamps of the files in your repository are consistently reset to the time of the commit they were last modified in after a clone/fetch.  `restore-mtime` is fast even on large repositories, capable of processing 100K files in less than 8 seconds, and works with submodules as well!

- Cleaning of submodules
  > The default GitHub checkout action has a long-standing open issue ([actions/checkout#358](https://github.com/actions/checkout/issues/358)) where it does not perform a `git clean` on any submodules.  Checkout Plus will automatically recursively iterate (as desired) into your repository's submodules and run `git clean` on them when both `clean: true` is set and `submodules` is set to either `true` or `recursive`.

# Quickstart

## New Builders

If you are new to GitHub Actions, simply create an actions .yaml, and add the following line to it:

```
- uses: coldfusionjp/ghaction-checkout-plus@v3
```

## Existing Checkout Action Builders

If you are already using GitHub's provided [actions/checkout](https://github.com/actions/checkout) action, simply change the following line in your actions .yaml:

```
- uses: actions/checkout@v3
```

to be:

```
- uses: coldfusionjp/ghaction-checkout-plus@v3
```

Any parameters which are passed to [actions/checkout](https://github.com/actions/checkout) can continue to be used as-is with Checkout Plus.

# Detailed Usage

<!-- start usage -->
```yaml
- uses: coldfusionjp/ghaction-checkout-plus@v3
  with:
    # Repository name with owner. For example, coldfusionjp/ghaction-checkout-plus
    # Default: ${{ github.repository }}
    repository: ''

    # The branch, tag or SHA to checkout. When checking out the repository that
    # triggered a workflow, this defaults to the reference or SHA for that event.
    # Otherwise, uses the default branch.
    ref: ''

    # Personal access token (PAT) used to fetch the repository. The PAT is configured
    # with the local git config, which enables your scripts to run authenticated git
    # commands. The post-job step removes the PAT.
    #
    # We recommend using a service account with the least permissions necessary. Also
    # when generating a new PAT, select the least scopes necessary.
    #
    # Learn more about creating and using encrypted secrets:
    # - https://help.github.com/en/actions/automating-your-workflow-with-github-actions/creating-and-using-encrypted-secrets
    #
    # Default: ${{ github.token }}
    token: ''

    # SSH key used to fetch the repository. The SSH key is configured with the local
    # git config, which enables your scripts to run authenticated git commands. The
    # post-job step removes the SSH key.
    #
    # We recommend using a service account with the least permissions necessary.
    #
    # Learn more about creating and using encrypted secrets:
    # - https://help.github.com/en/actions/automating-your-workflow-with-github-actions/creating-and-using-encrypted-secrets
    ssh-key: ''

    # Known hosts in addition to the user and global host key database. The public SSH
    # keys for a host may be obtained using the utility `ssh-keyscan`. For example,
    # `ssh-keyscan github.com`. The public key for github.com is always implicitly
    # added.
    ssh-known-hosts: ''

    # Whether to perform strict host key checking. When true, adds the options
    # `StrictHostKeyChecking=yes` and `CheckHostIP=no` to the SSH command line. Use
    # the input `ssh-known-hosts` to configure additional hosts.
    # Default: true
    ssh-strict: ''

    # Whether to configure the token or SSH key with the local git config
    # Default: true
    persist-credentials: ''

    # Relative path under $GITHUB_WORKSPACE to place the repository
    path: ''

    # Whether to execute `git clean -ffdx && git reset --hard HEAD` before fetching
    # Default: true
    clean: ''

    # Whether to restore the last modification time of the files in the repository
    # based on the date of the most recent commit that modified them.
    #
    # Enabling this feature is useful in CI/CD environments where files may be
    # incorrectly identified as being modified when they are not, preventing
    # incremental builds from working as expected. Git does not preserve the original
    # timestamp of committed files, and whenever repositories are cloned, or
    # branches/files are checked out, file timestamps are reset to the current date.
    # This feature will reset the timestamp of the file to the date of the most recent
    # commit where it was last modified, providing a consistent timestamp necessary
    # for incremental builds.
    #
    # Restoring the last modification time is also automatically performed for
    # submodules as desired, based on the submodules setting.
    #
    # In order to properly restore the last modification time of files in the
    # repository, the full history (a deep clone) is required. When `restore-mtime` is
    # enabled, `fetch-depth` will instead default to `0` to fetch the complete history
    # (however, this can be overridden if desired).
    #
    # Default: false
    restore-mtime: ''

    # Number of commits to fetch. 0 indicates all history for all branches and tags.
    # Default: 1
    fetch-depth: ''

    # Whether to download Git-LFS files
    # Default: false
    lfs: ''

    # Whether to checkout submodules: `true` to checkout submodules or `recursive` to
    # recursively checkout submodules.
    #
    # When the `ssh-key` input is not provided, SSH URLs beginning with
    # `git@github.com:` are converted to HTTPS.
    #
    # Default: false
    submodules: ''

    # Add repository path as safe.directory for Git global config by running `git
    # config --global --add safe.directory <path>`
    # Default: true
    set-safe-directory: ''

    # The base URL for the GitHub instance that you are trying to clone from, will use
    # environment defaults to fetch from the same instance that the workflow is
    # running from unless specified. Example URLs are https://github.com or
    # https://my-ghes-server.example.com
    github-server-url: ''
```
<!-- end usage -->

# Scenarios

- [Fetch all history for all tags and branches](#Fetch-all-history-for-all-tags-and-branches)
- [Checkout a different branch](#Checkout-a-different-branch)
- [Checkout HEAD^](#Checkout-HEAD)
- [Checkout multiple repos (side by side)](#Checkout-multiple-repos-side-by-side)
- [Checkout multiple repos (nested)](#Checkout-multiple-repos-nested)
- [Checkout multiple repos (private)](#Checkout-multiple-repos-private)
- [Checkout pull request HEAD commit instead of merge commit](#Checkout-pull-request-HEAD-commit-instead-of-merge-commit)
- [Checkout pull request on closed event](#Checkout-pull-request-on-closed-event)
- [Push a commit using the built-in token](#Push-a-commit-using-the-built-in-token)

## Fetch all history for all tags and branches

```yaml
- uses: coldfusionjp/ghaction-checkout-plus@v3
  with:
    fetch-depth: 0
```

## Checkout a different branch

```yaml
- uses: coldfusionjp/ghaction-checkout-plus@v3
  with:
    ref: my-branch
```

## Checkout HEAD^

```yaml
- uses: coldfusionjp/ghaction-checkout-plus@v3
  with:
    fetch-depth: 2
- run: git checkout HEAD^
```

## Checkout multiple repos (side by side)

```yaml
- name: Checkout
  uses: coldfusionjp/ghaction-checkout-plus@v3
  with:
    path: main

- name: Checkout tools repo
  uses: coldfusionjp/ghaction-checkout-plus@v3
  with:
    repository: my-org/my-tools
    path: my-tools
```
> - If your secondary repository is private you will need to add the option noted in [Checkout multiple repos (private)](#Checkout-multiple-repos-private)

## Checkout multiple repos (nested)

```yaml
- name: Checkout
  uses: coldfusionjp/ghaction-checkout-plus@v3

- name: Checkout tools repo
  uses: coldfusionjp/ghaction-checkout-plus@v3
  with:
    repository: my-org/my-tools
    path: my-tools
```
> - If your secondary repository is private you will need to add the option noted in [Checkout multiple repos (private)](#Checkout-multiple-repos-private)

## Checkout multiple repos (private)

```yaml
- name: Checkout
  uses: coldfusionjp/ghaction-checkout-plus@v3
  with:
    path: main

- name: Checkout private tools
  uses: coldfusionjp/ghaction-checkout-plus@v3
  with:
    repository: my-org/my-private-tools
    token: ${{ secrets.GH_PAT }} # `GH_PAT` is a secret that contains your PAT
    path: my-tools
```

> - `${{ github.token }}` is scoped to the current repository, so if you want to checkout a different repository that is private you will need to provide your own [PAT](https://help.github.com/en/github/authenticating-to-github/creating-a-personal-access-token-for-the-command-line).


## Checkout pull request HEAD commit instead of merge commit

```yaml
- uses: coldfusionjp/ghaction-checkout-plus@v3
  with:
    ref: ${{ github.event.pull_request.head.sha }}
```

## Checkout pull request on closed event

```yaml
on:
  pull_request:
    branches: [main]
    types: [opened, synchronize, closed]
jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - uses: coldfusionjp/ghaction-checkout-plus@v3
```

## Push a commit using the built-in token

```yaml
on: push
jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - uses: coldfusionjp/ghaction-checkout-plus@v3
      - run: |
          date > generated.txt
          git config user.name github-actions
          git config user.email github-actions@github.com
          git add .
          git commit -m "generated"
          git push
```

# License

While the scripts and documentation specific to this project are released under the [MIT License](LICENSE), this project integrates components from other projects which use different licenses.  Please see the included [NOTICE](NOTICE.md) for full details.
