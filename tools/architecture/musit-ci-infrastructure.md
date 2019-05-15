# MUSIT CI Infrastructure

All MUSIT projects are currently being built using GitLab CI. For details about GitLab CI please refer to the official [GitLab CI documentation](https://gitlab.com/help/ci/README.md). This goal of this document is to give an overview over how _MUSIT_ use this particular feature of GitLab.

## Build definitions

### MUSIT application - source code

Each project that needs to be built contains a file called `.gitlab-ci.yml` at their root directory. This is the "recipe" that tells GitLab CI what to do when a new CI build is triggered.

Currently we have 2 main CI build definitions. One for the backend services, and one for the frontend client. They are more or less defined in the same structure as below:

```yaml
######################################################################
# GitLab CI build script for the MUSIT backend project               #
######################################################################
# Defines some environment variables that should be made
# available to _all_ jobs in the CI pipeline.
variables:
  DOCKER_DRIVER: overlay
  # Setting specific folder for sbt-coursier to cache artifacts
  COURSIER_CACHE: "/root/cache/coursier"

# Defines the different stages for the CI pipeline. First the pipeline
# should execute the "test" stage, then attempt the "build" stage.
stages:
  - test
  - build
# Tells gitlab CI what to cache between pipeline runs
cache:
  untracked: true
  paths:
    - cache
# Definition for the "test" pipeline job. This is executed for every
# commit on all branches/forks.
test:
  # Tells the gitlab CI that this job should run in the "test" stage.
  stage: test
  # The tags section tells GitLabCI that the job needs a few prerequisites
  # to be able to run. In this case, the runner node should be "shared"
  # and must support "docker" builds.
  tags:
    - docker
    - shared
  # The image to use as execution environment for the "test" job.
  image: registry.gitlab.com/musit-norway/docker-scala-sbt
  # Here we define som environment variables to make available only for the
  # "test" job.
  variables:
    MUSIT_FUTURE_TIMEOUT: "15000"
    MUSIT_FUTURE_INTERVAL: "50"

  # This regex _should_ parse the console output of the job and extract the
  # code coverage result.
  coverage: '/\[info]\sAll\sdone\.\sCoverage\swas\s\[(.*)%\]/'
  # The actual script to execute for running the "test" job.
  script:
    - echo "Building and running tests..."
    # Check if the scalafmt modified any files during the build. If yes, fail the build.
    - sbt clean scalastyle scalafmt
    - git status
    - git diff --exit-code || (echo "ERROR Scala formatting check failed, see differences above."; false)
    - if [[ -n "$CODACY_PROJECT_TOKEN" ]]; then echo "Running with code coverage..."; sbt coverage test coverageReport; sbt coverageAggregate; sbt codacyCoverage; else echo "Coverage reporting disabled for forks"; sbt test; fi

# Definition for the "build" pipeline job. This is _only_ executed whenever a new
# commit arrives on the master branch of the upstream repository.
build:
  # Tells the gitlab CI that this job should run in the "build" stage.
  stage: build
  tags:
    - docker
    - musit
    - utv
  image: $MUSIT_DOCKER_REGISTRY/musit/docker-scala-sbt
  variables:
    # Since the "build" job requires docker to build the artifacts,
    # We need to ensure that the correct Docker driver is used.
    DOCKER_DRIVER: overlay
  
  # Here we define services that need to be present for the "build" to be
  # able to run. In this case we're telling the pipeline we want to run
  # an image allowing to execute "docker in docker".
  services:
    - $MUSIT_DOCKER_REGISTRY/library/docker:dind

  # The "before_script" configuration, allows for defining a preparation script.
  # Typically this is used to ensure all is set before the actual script runs.
  before_script:
    - echo "Running build $CI_JOB_ID for commit $CI_COMMIT_SHA with commit ref name $CI_COMMIT_REF_NAME"
    - mkdir $HOME/.docker
    - echo $DOCKER_AUTH_CONFIG > $HOME/.docker/config.json
    - docker info
  script:
    - sbt docker:publish
  # Here we tell the CI that this job should _only_ run for commits on the master
  # branch on the upstream repository.
  only:
    - master@MUSIT-Norway/musit

```

For the full definition please see the relevant repositories.

### Supporting docker images - used by source code builds

In addition to the build definitions for the source code repositories, we have a couple of git repositories for building docker images as well. These are specifically used for _executing_ the builds for our source code. There is one for building the backend, called [docker-scala-sbt](https://gitlab.com/MUSIT-Norway/docker-scala-sbt). And one for the frontend, called [docker-node](https://gitlab.com/MUSIT-Norway/docker-node). Pushing changes to these repositories will trigger a GitLab CI job that rebuilds the respective docker images, and pushes them up to the USIT internal docker registry (harbor).

The structure of these `.gitlab-ci.yml` files are different from the source code builds. But they follow the same basic structure as each other. Below is a sample script from the `docker-scala-sbt` repository:

```yaml
######################################################################
# GitLab CI build script for the docker-scala-sbt Docker image.      #
# The resulting image is used to build Scala based microservices     #
######################################################################
variables:
  DOCKER_DRIVER: overlay

stages:
  - build

build for gitlab registry:
  stage: build
  tags:
    - docker
    - shared
  image: docker:latest
  services:
    - docker:dind
  script:
    # Prepare the docker build context
    - cd gitlab-registry
    - mv ../play-scala .
    - mv ../sbtopts .
    # Build docker image and push to registry
    - docker login -u gitlab-ci-token -p $CI_JOB_TOKEN registry.gitlab.com
    - docker build -t registry.gitlab.com/musit-norway/docker-scala-sbt .
    - docker push registry.gitlab.com/musit-norway/docker-scala-sbt
  only:
    - master@MUSIT-Norway/docker-scala-sbt

build for harbor registry:
  stage: build
  tags:
    - docker
    - musit
  image: $MUSIT_DOCKER_REGISTRY/library/docker:dind
  services:
    - $MUSIT_DOCKER_REGISTRY/library/docker:dind
  before_script:
    - echo "Running build $CI_BUILD_ID"
    - mkdir $HOME/.docker
    - echo $DOCKER_AUTH_CONFIG > $HOME/.docker/config.json
  script:
    # Prepare the docker build context
    - cd harbor-registry
    - mv ../play-scala .
    - mv ../sbtopts .
    # Build docker image and push to registry
    - docker build -t $MUSIT_DOCKER_REGISTRY/musit/docker-scala-sbt:latest .
    - docker push $MUSIT_DOCKER_REGISTRY/musit/docker-scala-sbt:latest
  only:
    - master@MUSIT-Norway/docker-scala-sbt
```

## GitLab CI runner

The MUSIT project has some requirements with regards to how docker images are built. Specially concerning which docker images are used as the basis of _new_ docker images. The USIT internal policy states that docker images must be based off approved images in the internal docker registry (harbor). Which is a closed registry unavailable outside the USIT network. This means we have to ensure our builds are executed inside the USIT network boundries. To achieve that, we have installed an internal [gitlab-runner](https://docs.gitlab.com/runner/install/) on a server with access to the registry. This handles _all_ builds related to the MUSIT projects.
Do _not_ execute any builds on the GitLab CI Shared Runners.



