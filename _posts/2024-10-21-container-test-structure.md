---
layout: post
title: My overview of Container Structure Test
excerpt_separator: <!--preview-->
---
Recently I tested one tool called Container Structure Test and I wanted to shared my overview about it.
<!--preview-->

This is a framework to test container images. I found it in the Thoughtworks Technology [Radar](https://www.thoughtworks.com/en-es/radar/tools/container-structure-tests)
at the beginning of 2024. However, current edition does not include it.

![Thoughtworks Technology Radar]({{ site.baseurl }}/images/2024/container-structure/radar.jpg)

It caught my attention because I have never developed explicit tests for container images. Testing theory states that we should
have a large base of unit tests which takes low time to execute and provides fast feedback. Usually the container images are tested
implicitly in integration tests where the container is deployed and the functionality checked. However not all characteristics
of the image definition are asserted. For example, with a Dockerfile in mind, the application itself will be executed
and will allow the tests to pass if it works as expected, but there are other components like labels or users defined there
that might change and the application will continue the execution without further notice.

Having said that, let's take a look into this Container Structure Test tool. The repository
is available [here](https://github.com/GoogleContainerTools/container-structure-test). Google is the owner of it.
To execute the tool, one just need to run the following command, once installed:
```
   container-structure-test test --image gcr.io/registry/image:latest --config config.yaml

```
where gcr.io/registry/image:latest is the image selected in the example to be tested and config.yaml defines the different tests that the framework allows. These tests are the following:
- Command tests
- File existence tests
- File content tests
- Metadata tests
- License tests

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
This type of test checks whether a file or directory exists and its permissions.
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
This type permits testing different characteristics of the image. Actually they map to [Dockerfile](https://docs.docker.com/build/concepts/dockerfile/) keys, like the environment variables,
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
The selection of some options to run the container is available. From enviroment variables and files,
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

## Execution

Regarding the execution, the following image describes the CLI interface after executing some tests against a cached image.

![Execution example]({{ site.baseurl }}/images/2024/container-structure/example.JPG)

## Maintenance and support

Regarding the maintenance and support, which is something to consider when using a third party library in your projects, google is not officially
supporting it:

![Project is not supported]({{ site.baseurl }}/images/2024/container-structure/unsupported.jpg)

I was able to find more details about this decision, described here:

![Reason of end of project support]({{ site.baseurl }}/images/2024/container-structure/reason.jpg)

However, the project is actively maintained by the community. During this year, seven releases have been published so far.

## Conclusion

In conclusion, this library provides some benefits:
- Test all parts of your image definition that might not be covered (labels, env vars, ...)
- Provide quick feedback instead of waiting for integration tests to deploy your container
- Although some tests might look useless, it provides a safety net in case there are unforeseen changes (e.g. a change in a port done by error)
- It is an easy to learn framework
As drawbacks:
- Third party dependency. For me any new library needs to add value to your project. Otherwise you will need to keep vulnerabilities and updates and will cost time
- In my experience, mostly working with Dockerfiles, it is not changed that much, and most of its changes are just versions uplift.

It is interesting to know a tool to test container images. In my experience, I did not experience relevant bugs in the container definition, but could be useful for use cases like security where you have to define a specific user in the image, or basically to increase your trust in the application.
