[![test_with_pyre_config](https://github.com/facebook/pysa-action/actions/workflows/test_with_pyre_config.yml/badge.svg)](https://github.com/facebook/pysa-action/actions/workflows/test_with_pyre_config.yml)
[![test_no_pyre_config](https://github.com/facebook/pysa-action/actions/workflows/test_no_pyre_config.yml/badge.svg)](https://github.com/facebook/pysa-action/actions/workflows/test_no_pyre_config.yml)
[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](https://opensource.org/licenses/MIT)

# Pysa Github Action

<p align="center">
  <img src="https://raw.githubusercontent.com/facebook/pysa-action/main/logo.png">
</p>

[Python Static Analyzer (Pysa)](https://engineering.fb.com/2020/08/07/security/pysa/) is a security-focused static analysis tool that tracks flows of data from where they originate to where they terminate in a dangerous location. Pysa has been used to detect and disclose security issues on open source Python projects in the past, such as [CVE-2019-19775](https://cve.mitre.org/cgi-bin/cvename.cgi?name=CVE-2019-19775).

The Pysa GitHub Action enables you to run [Pysa](https://pyre-check.org/docs/pysa-basics/) in CI and view the results on [GitHub Security code scanning UI](https://docs.github.com/code-security/code-scanning/automatically-scanning-your-code-for-vulnerabilities-and-errors/managing-code-scanning-alerts-for-your-repository#viewing-the-alerts-for-a-repository).


## Usage
```yml
on:
  push:
    branches:
      - main
  pull_request:

name: Pysa

jobs:
  pysa:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v2

      - name: Run Pysa Action
        uses: facebook/pysa-action
        with:
          repo-directory: './'
          requirements-path: 'requirements.txt'
          infer-types: true
          include-default-sapp-filters: true
          sapp-filters-directory: path/to/custom-filters

```
See the [test workflows in this repository](.github/workflows) for more examples

## Dependencies

Pysa Action relies on [SAPP Action](https://github.com/facebook/sapp-action) to post process and filter Pysa results

## Inputs
### `repo-directory`

**Required**, Path to the python source code you want to analyze. If you want to analyze the root of your repo, use `'./'`. The default will be to analyze the root of your repository.

Since Pysa relies on [Pyre](https://github.com/facebook/pyre-check), Pysa Action will also look for a [`.pyre_configuration`](https://pyre-check.org/docs/configuration/#configuration-files) in the root of your `repo-directory`. If Pysa Action can't find a `.pyre_configuration` file in the root of your `repo-directory`, it will create the default Pyre configuration to use. If you notice any missing flows involving your project dependencies, it can be likely fixed by committing [the default `.pyre_configuration`](https://github.com/facebook/pysa-action/blob/main/action.yml#L105-L108) to your repo and updating the [`taint_models_path`](https://pyre-check.org/docs/pysa-running/#4-pysa-configuration) to point to where your dependencies are installed

### `requirements-path`

**Required**, Path to file containing your python code's dependencies relative to `repo-directory`. The default will look for [`requirements.txt`](https://pip.pypa.io/en/latest/reference/requirements-file-format/) in the root of the directory you specified in `repo-directory`.

Pysa Action will install all your project dependencies before the taint analysis stage and may miss flows for any dependencies not present in [`sys.path`](https://docs.python.org/3/library/sys.html#sys.path), so it is important to specify all your project dependencies in your `requirements.txt`

### `use-nightly`

When set to `true`, the action will use the [nightly version of Pysa](https://pypi.org/project/pyre-check-nightly/) to analyze your python code. The nightly version of Pysa tends to be unstable is not recommended you set this option to true unless you are adventurous. By default, the action will use the [latest stable version of Pysa](https://pypi.org/project/pyre-check/).

### `infer-types`

If this value is `true`, the action will run [`pyre infer`](https://pyre-check.org/docs/pysa-coverage/#pyre-infer) in-place to add type annotations to your python code. Unless your python code is sufficiently [type annotated](https://www.python.org/dev/peps/pep-0484/), it is highly recommended you set `infer-types` to `true`, since it'll greatly improve the quality and quantity of data flows Pysa is able to found.

Note that while viewing Pysa results, you may see that your source code has changed. Those changes are limited to your workflow run of Pysa and will not be committed to your repo. As a precaution to prevent confusion, the default for `infer-types` is false, however as mentioned earlier, it's strongly recommended you set `infer-types` to `true`.

### `sapp-version`

The version number of [SAPP](https://pypi.org/project/fb-sapp/) you would like to use to post process Pysa results. By default, the action will use the latest version of SAPP.

### `sapp-filters-directory`

Path relative to `repo-directory` where the SAPP filters you wrote that you want applied to filter the results of your Pysa runs are.

A description and guide to writing your own filters is available on the [SAPP Github Repo](https://github.com/facebook/sapp#filters). The description of what features are is available on the [Pysa documentation](https://pyre-check.org/docs/pysa-features/).

See the [`test/custom-filters`](test/custom-filters) in this repo for a example

### `include-default-sapp-filters`

When set to `true`, SAPP will filter your Pysa runs with the [default filters shipped with Pysa](https://github.com/facebook/pyre-check/tree/main/tools/sapp/pysa_filters). The SAPP filters shipped with Pysa are intended to filter out false positives even at the cost of [false negatives](https://pyre-check.org/docs/pysa-false-negatives/) to ensure Pysa results are as high signal as possible.

By default, Pysa Action will use the default SAPP filters to filter its results. There are a few use cases where you might want to set `include-default-sapp-filters` to `false`. For example:
- You prefer to apply no filters to your Pysa results, because you would like to see all Pysa results
- You prefer to filter Pysa results only using the SAPP filters you've written in `sapp-filters-directory`

## License

Pysa Action is licensed under the MIT license.
