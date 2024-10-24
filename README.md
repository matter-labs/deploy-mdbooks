# ZKsync Era: Github Action to deploy mdBook to GitHub Pages.

[![Logo](eraLogo.png)](https://zksync.io/)

ZKsync Era is a layer 2 rollup that uses zero-knowledge proofs to scale Ethereum without compromising on security or
decentralization. Since it's EVM compatible (Solidity/Vyper), 99% of Ethereum projects can redeploy without refactoring
or re-auditing a single line of code. ZKsync Era also uses an LLVM-based compiler that will eventually let developers
write smart contracts in C++, Rust and other popular languages.

## Overview

This GitHub Action automates the process of building, testing, and deploying a versioned [mdBook](https://github.com/rust-lang/mdBook) project to GitHub Pages. The action supports multiple versions of the documentation, enabling versioned documentation for your project.

## Features

- **Automatic Deployment**: Automatically deploys your mdBook documentation to the `gh-pages` branch.
- **Version Control**: Deploys different versions of your documentation based on the branch or custom input.
- **Testing Support**: Optionally runs `mdbook test` before deploying to ensure the book is functional.
- **Customizable**: Supports custom mdBook versions, output directories, and more.

## Example Workflow

Here is a simple example for deploying mdBook documentation to GitHub Pages:

```yaml
name: Deploy docs

on:
  push:
    branches:
      - "main"

jobs:

  deploy-docs:
    runs-on: ubuntu-latest
    permissions:
      contents: write
    steps:
      - name: Checkout
        uses: actions/checkout@v4

      - name: Deploy
        uses: matter-labs/deploy-mdbooks@v1
        with:
          version: ${{ inputs.version || github.ref_name }}
          docs-dir: ${{ inputs.docs-dir || 'docs' }}
          github-token: ${{ secrets.GITHUB_TOKEN }}
```


## More Complex Workflow

Here's a more complete example for deploying versioned mdBook documentation from both pushes to the default branch and tags:

```yaml
name: Deploy docs

on:
  push:
    branches:
      - "main"
    tags:
      - "*.*.*"
      - "v*.*.*"
    paths:
      - 'docs/**'
      - '.github/workflows/deploy-docs.yaml'
  workflow_dispatch:
    inputs:
      version:
        type: string
        description: "Version of the documentation to deploy"
        required: false
        default: "latest"
      docs-dir:
        type: string
        description: "Directory with the documentation"
        required: false
        default: "docs"

# Allow only one concurrent deployment, skipping runs queued between the run in-progress and latest queued.
# However, do NOT cancel in-progress runs as we want to allow these production deployments to complete.
concurrency:
  group: "pages"
  cancel-in-progress: false

jobs:

  deploy-docs:
    runs-on: ubuntu-latest
    permissions:
      contents: write
    steps:
      - name: Checkout
        uses: actions/checkout@v4

      - name: Deploy
        uses: matter-labs/deploy-mdbooks@v1
        with:
          version: ${{ inputs.version || github.ref_name }}
          docs-dir: ${{ inputs.docs-dir || 'docs' }}
          github-token: ${{ secrets.GITHUB_TOKEN }}
          force-orphan: true
          enable-tests: true
          mdbook-version: "v0.4.40"
```

## Inputs

| Name               | Type      | Required | Default       | Description                                                                 |
|--------------------|-----------|----------|---------------|-----------------------------------------------------------------------------|
| `github-token`     | `string`  | ✅        | N/A           | GitHub token used to authenticate and push to `gh-pages` branch.            |
| `version`          | `string`  | ❌        | `latest`      | The version of the documentation to deploy. Defaults to "latest".           |
| `docs-dir`         | `string`  | ❌        | `docs`        | The directory containing the mdBook source files.                           |
| `mdbook-version`   | `string`  | ❌        | `latest`      | The version of mdBook to use. Default is the latest release.                |
| `enable-tests`     | `boolean` | ❌        | `true`        | Enable or disable `mdbook test`.                                            |
| `force-orphan`     | `boolean` | ❌        | `false`       | Force `gh-pages` branch to be orphan (do not keep history of commits).      |
| `publish-branch`   | `string`  | ❌        | `gh-pages`    | The branch to publish documenation to.                                      |

## Outputs

- The action pushes the built documentation to the `gh-pages` branch in a directory corresponding to the specified version.

## Notes

- Action is supported on Linux only.
- If the `gh-pages` branch does not exist, it will be created automatically.
- Make sure that GitHub Pages is set to deploy from the `gh-pages` branch in your repository settings.

## License

This project is distributed under the terms of either:
- Apache License, Version 2.0, ([LICENSE-APACHE](LICENSE-APACHE) or <http://www.apache.org/licenses/LICENSE-2.0>)
- MIT license ([LICENSE-MIT](LICENSE-MIT) or <https://opensource.org/blog/license/mit/>)
