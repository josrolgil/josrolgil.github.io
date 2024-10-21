---
layout: post
title: Container Structure Test: my overview
---
Recently I tested one tool called Container Structure Test and I wanted to shared my overview about it.

This is a tool to test container images. I found it in the Thoughtworks Technology [Radar](https://www.thoughtworks.com/en-es/radar/tools/container-structure-tests)
at the beginning of 2024. However, current edition does not include it.

![_config.yml]({{ site.baseurl }}/images/2024/container-structure/radar.JPG)

It caught my attention because we do not have unit tests for container images. Testing theory states that we should
have a large base of unit tests that execute quick to provide fast feedback. Usually the container images are tested
implicitly in integration tests where the container is deployed and the functionality checked. Even not all characteristics
of the image definition are asserted. For example, with a Dockerfile in mind, the application itself will be executed
and will allow the tests to pass if it works as expected, but there are other components like labels or users defined there
that might change and the application will continue the execution without further changes.

Having said that, let's take a look into this Container Structure Test tool. The [repository] https://github.com/GoogleContainerTools/container-structure-test
is available [here]. Google is the owner of it
To execute the tool, one just need to run the following command, once installed
```
   container-structure-test test --image gcr.io/registry/image:latest --config config.yaml
```
where config.yaml defines the different tests that the framework allows.
These are command tests, file existence tests, file content tests, metadata tests and licence tests.
## Command tests
With command tests you can execute a command and expect an output. You can set arguments, a setup command, exclussions...
For example:
```
commandTests:
  - name: "gunicorn flask"
    setup: [["virtualenv", "/env"], ["pip", "install", "gunicorn", "flask"]]
    teardown: [echo “done”]
    command: "which"
    args: ["gunicorn"]
    expectedOutput: ["/env/bin/gunicorn"]
  - name:  "apt-get upgrade"
    command: "apt-get"
    args: ["-qqs", "upgrade"]
    excludedOutput: [".*Inst.*Security.* | .*Security.*Inst.*"]
    excludedError: [".*Inst.*Security.* | .*Security.*Inst.*"]
```

## File existence tests
This type of test checks whether a file or directory exists and its its permissions.
For example:
```
  fileExistenceTests:
  - name: 'Root'
    path: '/'
    shouldExist: true
    permissions: '-rw-r--r--'
    uid: 1000
    gid: 1000
    isExecutableBy: 'group'

```

## File content tests
This type of test reads the content of a file or directory and expects or excludes
some content as string.
For example:
```
  fileContentTests:
  - name: 'Debian Sources'
    path: '/etc/apt/sources.list'
    expectedContents: ['.*httpredir\.debian\.org.*']
    excludedContents: ['.*gce_debian_mirror.*']
```

## Metadata test
This type permits testing different keys of the dockerfile, like the environment variables,
the labels, the ports, volumes, commands...
An example:
```
metadataTest:
  envVars:
    - key: foo
      value: baz
  labels:
    - key: 'com.example.vendor'
      value: 'ACME Incorporated'
    - key: 'build-date'
      value: '^\d{4}-\d{2}-\d{2}T\d{2}:\d{2}:\d{2}\.\d{6}$'
      isRegex: true
  exposedPorts: ["8080", "2345"]
  volumes: ["/test"]
  entrypoint: []
  cmd: ["/bin/bash"]
  workdir: "/app"
  user: "luke"
```

## License test
This only checks debian licenses or some files given. This type of tests seems to be
fulfilling a requirement at Google and not very flexible.
An example:
```
licenseTests:
- debian: true
  files: ["/foo/bar", "/baz/bat"]
```

## Options
The selection of some options to run the container is available. From enviroment variable and file,
the use of tty or which priviledge mode. This could be useful to simulate different execution scenarios.
For example:
```
containerRunOptions:
  user: "root"                  # set the --user/-u flag
  privileged: true              # set the --privileged flag (default: false)
  allocateTty: true             # set the --tty flag (default: false)
  envFile: path/to/.env         # load environment variables from file and pass to container (equivalent to --env-file)
  envVars:                      # if not empty, read each envVar from the environment and pass to test (equivalent to --env/e)
    - SECRET_KEY_FOO
    - OTHER_SECRET_BAR
  capabilities:                 # Add list of Linux capabilities (--cap-add)
    - NET_BIND_SERVICE
  bindMounts:                   # Bind mount a volume (--volume, -v)
    - /etc/example/dir:/etc/dir
globalEnvVars:
      - key: "VIRTUAL_ENV"
        value: "/env"
      - key: "PATH"
        value: "/env/bin:$PATH"
```
Therefore, with these five type of test you can cover the image.

Regarding the execution, the following image describes the CLI interface after executing some tests agains a cached image.

![_config.yml]({{ site.baseurl }}/images/2024/container-structure/example.JPG)

Regarding the maintenance and support, which is something to consider when using a third party library in your projects, google is not officially
supporting it.

![_config.yml]({{ site.baseurl }}/images/2024/container-structure/unsupported.JPG)

I was able to find more details about this decision, described here

![_config.yml]({{ site.baseurl }}/images/2024/container-structure/reason.JPG)

However, the project is actively maintained by the community. These years five releases have been published.

In conclussion, this library provides some benefits:
- Test all parts of your image definition that might not be covered (labels, env vars, ...)
- Provide quick feedback instead of waiting for integration tests to deploy your container
- Although some tests might look useless, it provides a safety net in case there are unforeseen changes (e.g. a change in a port done by error)
- It is an easy to learn framework
As drawbacks:
- Third party dependency. For me any new library needs to add value to your project. Otherwise you will need to keep vulnerabilities and updates and will cost time
- In my experience, mostly working with Dockerfiles, it is not changed that much, and most of its changes are just versions uplift.
