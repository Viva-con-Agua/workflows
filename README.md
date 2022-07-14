# Reusable workflow definitions 

This repository contains [reusable workflows](https://docs.github.com/en/actions/using-workflows/reusing-workflows) for GitHub Actions that bundle common tasks of our CI/CD pipeline.

The contained workflows are:
- [build_image.yml](.github/workflows/build_image.yml) which builds a container and pushes it to GitHub packages.
- [deploy.yml](.github/workflows/deploy.yml) for setting the deployed image digest in a `-deploy` repository.
    The actual deployment is expected to be done by *ArgoCD* which should use the `-deploy` repository as source.
