# hub-user-image-template :paperclip:

This is a template repository for creating dedicated user images for UC Berkeley hubs.

## Overall workflow :gear:

The overall workflow is to:

1. Fork this repository to create your image repository

2. Hook your image repository to Google Artifact Registry

3. Customize the image by editing repo2docker files in your image repository.

   Changes can either be done by direct commits to main on your image repository, or through a pull request from a fork of your image repository. Direct commits will build the image and push it to GAR. PRs will build the image and offer a link to test it using Binder (currently disabled). Merging the PR will cause a commit on main and therefore trigger a build and push to GAR.

4. Configure your Hub to use this new image

### In-depth guide

Checkout the 2i2c docs for an in-depth guide on how to use this template repository to create a custom user image and use it for your hub :arrow_right: https://infrastructure.2i2c.org/howto/update-env/#split-up-an-image-for-use-with-the-repo2docker-action.

Here's a rough guide on how to create your own fresh user image :arrow_right: https://docs.datahub.berkeley.edu/en/latest/admins/howto/new-image.html.

After forking this repo and bringing in the commit history (if any) of the image, you will need to set two [Github Actions Repository Variables](https://docs.github.com/en/actions/learn-github-actions/variables) for the image:  `HUB` and `IMAGE`.

`HUB` is the short name of the hub (eg: `data100`, `datahub`, etc).
`IMAGE` is the path to the image in the Artifact Registry (eg: `ucb-datahub-2018/user-images/<hubname>-user-image`)

## About this template repository :information_source:

This template repository enables [jupyterhub/repo2docker-action](https://github.com/jupyterhub/repo2docker-action).
This GitHub action builds a Docker image using the contents of this repo and pushes it to the [Google Artifact Registry](https://cloud.google.com/artifact-registry) registry.

### The environment

It provides an example of a `environment.yml` conda configuration file for repo2docker to use.
This file can be used to list all the conda packages that need to be installed by `repo2docker` in your environment.
The `repo2docker-action` will update the [base repo2docker](https://github.com/jupyterhub/repo2docker/blob/HEAD/repo2docker/buildpacks/conda/environment.yml) conda environment with the packages listed in this `environment.yml` file.

**Note:**
A complete list of possible configuration files that can be added to the repository and be used by repo2docker to build the Docker image, can be found in the [repo2docker docs](https://repo2docker.readthedocs.io/en/latest/config_files.html#configuration-files).

### The GitHub Action workflows

This template repository provides some GitHub Action workflows that can build and push the image to Google Artifact Repository when configured, and test the image on Binder.

![Workflows](images/workflows.png)

#### 1. Build and test container image :arrow_right: [test.yaml](https://github.com/berkeley-dsep-infra/hub-user-image-template/blob/main/.github/workflows/test.yaml)

This workflow is triggered when a Pull Request is opened against the default branch (`main`)..
During PR builds, the image is **only** built and **not** pushed, unless explicitly configured to do so.

#### 2. Test this PR on Binder Badge :arrow_right: [binder.yaml](https://github.com/berkeley-dsep-infra/hub-user-image-template/blob/main/.github/workflows/binder.yaml.disable)

*Temporarily disabled*

Since our images are typically large and take > 10m to build, our Binder builds will currently time out.

This workflow posts a comment inside a pull request, every time a pull request gets opened. The comment contains a "Test this PR on Binder" badge, which can be used to access the image defined by the PR in [mybinder.org](https://mybinder.org/).

![Test this PR on Binder](images/binder-badge.png)

#### 3. Build, test and push container image :arrow_right: [build-push-open-pr.yaml](https://github.com/berkeley-dsep-infra/hub-user-image-template/blob/main/.github/workflows/build-push-open-pr.yaml)

After a PR is merged to `main`, this workflow builds the image again, pushes to the Artifact Registry and will create a push to the [Datahub repo](https://github.com/berkeley-dsep-infra/datahub) to update the image tag for any hubs that use this image.  The PR there will need to be created manually.