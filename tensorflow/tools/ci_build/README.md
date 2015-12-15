# TensorFlow.org Continuous Integration

This directory contains all the files and setup instructions to run
continuous integration [ci.tensorflow.org](http://ci.tensorflow.org).



## How it works

We use [jenkins](https://jenkins-ci.org/) as our continuous integration.
It is running at [ci.tensorflow.org](http://ci.tensorflow.org).
All the jobs are run within [docker](http://www.docker.com/) containers.

Builds can be triggered by push to master, push a change set or manually.
The build started in jenkins will first pull the git tree (with all submodules).
Then jenkins builds a docker container (using one of those Dockerfile.*
files in this directory). The build itself is run within the container itself.

Source tree lives in jenkins job workspace. Docker container for jenkins
are transient - deleted after the build. Containers build very fast thanks
to docker caching. Individual builds are fast thanks to bazel caching.



## Implementation Details

* `bazel-ci_build-cache` directory is mounted to docker container as a home
  directory of user performing the build. It is mounted using docker's --volume
  parameter. This way we cache bazel output between builds.

* [builds](builds) directory within this folder contains shell scripts to run
  inside the container. They essentially contains workarounds for current
  limitations of bazel.

  - [builds/configured](builds/configured) is a wrapper script automating
      [./configure](../../../configure)

  - [builds/with_user](builds/with_user) is a wrapper script creating inside
    container the same user and group as the user (outside the container)
    running the [ci_build.sh](ci_build.sh).

  - The other scripts in [builds](builds) are bazel workarounds and multi-line
    builds.



## Run It Yourself

1. Install [Docker](http://www.docker.com/). Follow instructions
   [on the Docker site](https://docs.docker.com/installation/).

2. Clone tensorflow repository.

   ```bash
git clone https://github.com/tensorflow/tensorflow.git
```

3. Go to tensorflow directory

   ```bash
cd tensorflow
```

4. Build what you want, for example

   ```bash
tensorflow/tools/ci_build/ci_build.sh CPU bazel test //tensorflow/...
```

Note: You can run it without docker using
`tensorflow/tools/ci_build/builds/configured CPU bazel test //tensorflow/...`
and if you run [./configure](../../../configure) yourself you are left with
only `bazel test //tensorflow/...` to run.



## Jobs

The jobs run by [ci.tensorflow.org](http://ci.tensorflow.org) include following:

```bash
# Note: You can run the following one-liners yourself if you have Docker.

# build and run cpu tests
tensorflow/tools/ci_build/ci_build.sh CPU bazel test //tensorflow/...

# build gpu
tensorflow/tools/ci_build/ci_build.sh GPU bazel build -c opt --config=cuda //tensorflow/...

# build pip with gpu support
tensorflow/tools/ci_build/ci_build.sh GPU tensorflow/tools/ci_build/builds/gpu_pip.sh

# build android example app
tensorflow/tools/ci_build/ci_build.sh ANDROID tensorflow/tools/ci_build/builds/android.sh
```

**Note**: The set of jobs and how they are triggered is still evolving.
There are builds for master branch on cpu, gpu and android. There is a build
for incoming gerrit changes. Gpu tests and benchmark are coming soon. Check
[ci.tensorflow.org](http://ci.tensorflow.org) for current jobs.
