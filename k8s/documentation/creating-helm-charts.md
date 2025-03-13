# Creating a Helm Chart Repository in GitHub
This guide explains how to create a Helm chart repository in GitHub, including the structure of the repository, the purpose of each file, and a GitHub Actions pipeline to release Helm chart packages automatically.

By following this guide, you can create a GitHub-hosted Helm chart repository and automate the release process using GitHub Actions. This setup ensures that any updates to the main branch automatically generate new Helm chart releases.

## [Helm](https://helm.sh/)
- Package manager for Kubernetes
- Helm Charts help you manage (define, install, upgrade) complex Kubernetes apps
- Charts are Kubernetes packages, a bundle of information necessary to create an instance of a Kubernetes app
- Easy to create, version, share, and publish deployment configs
- Configs for the k8s objects can be merged into a packaged chart to create a releasable object for an automatic deployment pipeline


## Create a GitHub repository and a releasable object 
Example repository for the semlookp-ui deployment: https://github.com/zbmed/semlookp-deployment/tree/main/k8s/semlookp-ui

The repository consists of a k8s directory with project specific folders for the deployment. Our typical Helm chart repositories consist of the following files and folders:

```
my-helm-repo/
├── templates/
│   ├── deployment.yaml
│   ├── service.yaml
│   ├── ingress.yaml
│   └── other-manifest-files.yaml
├── .helmignore
├── Chart.yaml
├── values.yaml
```

- **templates/**: This folder contains Kubernetes manifest files. These files define the resources that Helm will deploy.
- **.helmignore**: Specifies files and directories to be ignored when packaging the Helm chart (similar to `.gitignore`).
- **Chart.yaml**: Contains metadata about the Helm chart, including its name, version, description, and maintainers. IMPORTANT: You must manually increase the version to trigger a new release!
- **values.yaml**: Defines the default configuration values for the chart, which users can override when installing or upgrading the chart.

## Setting Up a GitHub Actions Pipeline for Chart Releases

To automate the release of Helm charts, we use GitHub Actions with the following workflow. 

For a successful run, i) an empty gh-pages branch must exist, ii) the GitHub repository must be public, and iii) GitHub Pages must be enabled in the project settings.

### GitHub Actions Workflow: `release.yml`

```yaml
name: Release Charts

on:
  push:
    branches:
      - main

jobs:
  release:
    runs-on: ubuntu-latest
    steps:
      - name: Checkout
        uses: actions/checkout@v3
        with:
          fetch-depth: 0

      - name: Configure Git
        run: |
          git config user.name "$GITHUB_ACTOR"
          git config user.email "$GITHUB_ACTOR@users.noreply.github.com"

      - name: Run chart-releaser
        uses: helm/chart-releaser-action@v1.5.0
        with:
          charts_dir: k8s
        env:
          CR_TOKEN: "${{ secrets.GITHUB_TOKEN }}"
```

When a new Helm chart version is released, an index.yaml file is generated and published on the gh-pages branch. This file is essential for Helm to recognize and fetch chart versions. It contains metadata about the available charts.

#### How This Works

**Chart Packaging**: The Helm chart is packaged into a `.tgz` archive and stored in GitHub Releases.

**Index Generation**: The `chart-releaser-action` updates `index.yaml` with the new chart version.

**Publishing to `gh-pages`**: The updated `index.yaml` is pushed to the `gh-pages` branch, making it publicly available.

## Using the Helm Chart

Once the `gh-pages` branch hosts `index.yaml`, users can add the Helm repo and install the chart (e.g. semlookp-ui):

```sh
helm repo add semlookp-deployment https://zbmed.github.io/semlookp-deployment/
helm repo update
helm install semlookp-ui semlookp-deployment/semlookp-ui --version 0.0.3
```

If the chart is not published, it can be used locally with
```sh
helm install semlookp-ui path-to-the-project-root/semlookp-deployment/semlookp-ui
```