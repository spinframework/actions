# `spinframework/actions` - GitHub Action collection for SpinFramework


With the `spinframework/actions` collection, you can incorporate [Spin](https://spinframework.dev/v3/index) in your [GitHub Action](https://help.github.com/en/actions/automating-your-workflow-with-github-actions/configuring-a-workflow). 

This collection of Actions enables the following use cases:

- [x] set up Spin CLI and plugins using [`spinframework/actions/spin/setup@v1`](#install-spin-cli-and-plugins---spinframeworkactionsspinsetupv1)
- [x] build and push your Spin app to an OCI registry using [`spinframework/actions/spin/push@v1`](#push-spin-app-to-a-registry---spinframeworkactionsspinpushv1)


Let's take a look at each one to learn about the required inputs and walk through an example. 

## Install Spin CLI and Plugins - `spinframework/actions/spin/setup@v1`

setup `spin` with optional plugins

### Inputs

| Name         | Required | Default | Description                                                                                                                                 |
| ------------ | -------- | ------- | ------------------------------------------------------------------------------------------------------------------------------------------- |
| version      | False    | latest  | The version of `spin` to install.                                                                                                           |
| plugins      | False    | -       | The comma-separated list of Spin plugins to install. [Learn more about Spin plugins.](https://spinframework.dev/v3/managing-plugins)  |
| github_token | False    | -       | The GitHub token for querying/downloading `spin` releases. If provided, it avoids GitHub API rate limiting during GitHub actions executions |

### Examples

#### Setting up latest version of `spin` 

```yaml
name: spin

on:
  - push

jobs:
  spin:
    runs-on: ubuntu-latest
    name: Setup spin
    steps:
      - name: Setup `spin`
        uses: spinframework/actions/spin/setup@v1

      - name: Run `spin version`
        run: "spin --version"
```

#### Setting up a specific version of `spin` 

```yaml
name: spin

on:
  - push

jobs:
  spin:
    runs-on: ubuntu-latest
    name: Setup spin
    steps:
      - name: Setup `spin`
        uses: spinframework/actions/spin/setup@v1
        with:
          version: "v0.10.1"

      - name: Run `spin version`
        run: "spin --version"
```

#### Setting up `spin` along with additional plugins

```yaml
name: spin

on:
  - push

jobs:
  spin:
    runs-on: ubuntu-latest
    name: Setup spin
    steps:
      - name: Setup `spin`
        uses: spinframework/actions/spin/setup@v1
        with:
          version: "v0.10.1"
          plugins: js2wasm

      - name: Run `spin version`
        run: "spin --version"
```

## Push Spin app to a Registry - `spinframework/actions/spin/push@v1`

Build and push the Spin app to your desired OCI Registry (note that this registry must have a publicly accessible endpoint). Also note this action has a prerequisite on Spin already being installed. 

### Inputs

| Name               | Required | Default   | Description                                                                              |
| ------------------ | -------- | --------- | ---------------------------------------------------------------------------------------- |
| registry_reference | True     | -         | The registry and reference to publish the app to e.g. ttl.sh/cloud-start:v0.0.1 |
| manifest_file      | False    | spin.toml | Path to `spin.toml`                                                                      |
| registry           | False    | -         | if provided, used to login to OCI Registry                                               |
| registry_username  | False    | -         | if provided, used to login to OCI Registry                                               |
| registry_password  | False    | -         | if provided, used to login to OCI Registry                                               |

### Outputs

| Name   | Description                                         |
| ------ | --------------------------------------------------- |
| digest | The image digest of the pushed app e.g. sha256:...  |

### Example

```yaml
name: spin

on:
  - push

jobs:
  spin:
    runs-on: ubuntu-latest
    name: Build and push
    steps:
      - uses: actions/checkout@v4

      - name: Setup `spin`
        uses: spinframework/actions/spin/setup@v1
        with:
          version: "v0.10.1"
          plugins: js2wasm

      - name: build and push
        id: push
        uses: spinframework/actions/spin/push@v1
        with:
          registry: ghcr.io
          registry_username: ${{ github.actor }}
          registry_password: ${{ secrets.GITHUB_TOKEN }}
          registry_reference: "ghcr.io/${{ env.REPOSITORY }}/${{ env.SAMPLE_APP_IMAGE_NAME }}:${{ github.run_id }}-2"
          manifest_file: example-app/spin.toml

      - name: echo digest
        run: echo ${{ steps.push.outputs.digest }}

```
