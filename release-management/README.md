# Release Management

In this repository you will find a few templates to help get your applications deployed.

## gitlab-ci/Dev-Pipeline.yml

`Dev-Pipeline.yml` contains everything you need to get your application up and running in a dev environment, with all of the necessary steps already built into the pipeline. It also allows you to trigger downstream jobs to promote your code (with the help of `Stage-Promotion-Pipeline.yml`) to any environments down the line.

To begin, ensure you have a `Dockerfile` at the root of your repository, and it is also recommended to have your own helm chart at `./chart`. 

Then just create a `.gitlab-ci.yml` file in your development repository, that imports `ci.yaml` and configures your deployment with the following code:

```
production:
  environment:
    url: https://<DEV_APP_URL>

include:
  project: dev/ep/release-management
  file: /Jobs/EP-AutoDevops-Template.yml

```

To trigger a downstream job, you can add it with the following code:

```
trigger <ENVIRONMENT_NAME>:
  stage: trigger
  trigger:
    project: <GITLAB>/<GROUP>/<PROJECT>
    strategy: depend
```

## gitlab-ci/Stage-Promotion-Pipeline.yml
`Stage-Promotion-Pipeline.yml` helps promote your application to subsequent stages, once your dev deployment has passed. Simply create a new repository in a group attached to the cluster you want to deploy to, and add the following `.gitlab-ci.yml` which imports and extends `cd.yaml`:

```
production_manual:
  environment:
    name: <ENVIRONMENT_NAME>
    url: https://<NEW_ENV_APP_URL>
  needs:
    - project: <GITLAB>/<GROUP>/<PROJECT> # the project where we set up the dev pipeline
      job: production
      ref: master
      artifacts: true

include:
  project: dev/ep/release-management
  file: /Jobs/EP-Stage-Promotion-Template.yml
```
