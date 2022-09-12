# Reusable workflow definitions

This repository contains [reusable workflows](https://docs.github.com/en/actions/using-workflows/reusing-workflows) for
GitHub Actions that bundle common tasks of our CI/CD pipeline.

The contained workflows are:

- [build_image.yml](.github/workflows/build_image.yml) which builds a container and pushes it to GitHub packages.
- [deploy.yml](.github/workflows/deploy.yml) for setting the deployed image digest in a `-deploy` repository

  The deployment configuration is expected to be based on [kustomize](https://kustomize.io/) with a `kustomization.yml` file in the folders `stage` and `deploy`.
  This workflow then updates the deployed image digest in the `kustomization.yml` file corresponding to the current branch (i.e. prod for main and stage for stage).

  The actual deployment based on the updated `kustomization.yml` is expected to be done by *ArgoCD* which should use the `-deploy` repository as source.

- [check_pre_commit.yml](.github/workflows/check_pre_commit.yml) for checking whether all [pre-commit](https://pre-commit.com/) checks pass

See the linked workflow definitions for the list of inputs that they expect.

## How To Use

To use one of the defined workflows, define the job in your repository like this.
Also refer to the [GitHub Reference](https://docs.github.com/en/actions/using-workflows/reusing-workflows#calling-a-reusable-workflow=) for calling reusable workflows. 

```yaml
# .github/workflows/my_awesome_workflow.yml
…
jobs:
  my-awesome-job:
    uses: Viva-con-Agua/workflows/.github/workflows/build_image.yml@main
    with:
      image_tag: v42.0.7
```

## CI/CD Example

For example, a fully functioning *CI/CD* pipeline could look like this, assuming that the following repositories exist
and have the described content.

| GitHub Repo                 | Necessary Content                                                                                                            |
|-----------------------------|------------------------------------------------------------------------------------------------------------------------------|
| Viva-con-Agua/my-app        | - The workflow definition below<br/>- A Dockerfile that builds the app                                                       |
| Viva-con-Agua/my-app-deploy | - A `prod` directory with kustomize based deployment config<br/>- A `stage` directory with kustomize based deployment config |

```yaml
# .github/workflows/container_cd.yml
name: container_cd
on:
  push:
    branches:
      - main
      - stage

jobs:
  build-container:
    uses: Viva-con-Agua/workflows/.github/workflows/build_image.yml@main
  deploy-container:
    needs: [ build-container ]
    uses: Viva-con-Agua/workflows/.github/workflows/deploy.yml@main
    secrets: inherit
    with:
      image_name: ${{ needs.build-container.outputs.image_name }}
      new_digest: ${{ needs.build-container.outputs.image_digest }}
```
