# Debugging AWS Lambda with VSCode

Here the resources to the webinar.

## Prerequisites

These are the things needed to set up a AWS SAM project for local debugging.

- [Docker](https://www.docker.com/get-started)
- [AWS Account](https://aws.amazon.com/premiumsupport/knowledge-center/create-and-activate-aws-account/)
- [AWS CLI](https://docs.aws.amazon.com/cli/latest/userguide/cli-chap-install.html)
- [AWS SAM CLI](https://docs.aws.amazon.com/serverless-application-model/latest/developerguide/serverless-sam-cli-install.html)
- [VSCode](https://code.visualstudio.com/)

## Setup

The project is set up with the help of the AWS SAM CLI.

    sam init

This command will create an AWS SAM project with a basic `hello-world` function that uses the Node.js v10 runtime. The project will be created inside a new `sam-app` directory.

## Run Lambda Locally

The AWS SAM CLI allows us to run a AWS Lambda function locally inside a Docker container.

### Without Debugger

A basic execution can be done with the following command:

    sam local invoke -e event.json HelloWorldFunction

### With Debugger

A debug execution can be done with the following command:

    sam local invoke -d 9999 -e event.json HelloWorldFunction

The port here has to be the same as in the `launch.json`

## VSCode launch.json Configurations

The configurations used inside the `launch.json`

### Basic Debug

The basic launch config will connect to an already running AWS SAM CLI process. So SAM CLI has to be run manually first.

```json
{
  "name": "Attach to SAM CLI",
  "type": "node",
  "request": "attach",
  "address": "localhost",
  "port": 9999,
  "localRoot": "${fileDirname}",
  "remoteRoot": "/var/task",
  "protocol": "inspector",
  "stopOnEntry": false
}
```

### VSCode Integration Debug

This config allows a one-click experience, but has to be created for every function/event combination. VSCode will start the SAM CLI by itself.

```json
{
  "name": "Debug with event.json",
  "type": "node",
  "request": "launch",
  "runtimeExecutable": "sam",
  "runtimeArgs": [
    "local",
    "invoke",
    "-d",
    "9999",
    "-t",
    "${workspaceRoot}/template.yaml",
    "-e",
    "${workspaceRoot}/events/event.json",
    "${selectedText}"
  ],
  "port": 9999,
  "localRoot": "${fileDirname}",
  "remoteRoot": "/var/task",
  "protocol": "inspector",
  "stopOnEntry": false
}
```

### Advanced VSCode Integration Debug

This will pass the selected text, which should contain of function name and a event name, to a special shell script. This script will, in turn, pass it to the SAM CLI as arguments. This lets VSCode start SAM CLI and allows for one launch config for multiple events and functions.

The `launch.json` config looks like this:

```json
{
  "name": "Debug with script",
  "type": "node",
  "request": "launch",
  "runtimeExecutable": "${workspaceRoot}/debug.sh",
  "runtimeArgs": ["${selectedText}"],
  "port": 9999,
  "localRoot": "${fileDirname}",
  "remoteRoot": "/var/task",
  "protocol": "inspector",
  "stopOnEntry": false
}
```

It will pass the `selectedText` as an argument to the `debug.sh`

The `debug.sh` looks like this:

```bash
#!/bin/bash
IFS=' ' read -r -a array <<< "$1"

sam local invoke -d 9999 \
-t "./template.yaml"
-e "./events/${array[1]}.json" \
${array[0]}
```

**NOTE:** Don't forget to set execution permissions for the `debug.sh`! :D

It will split the `selectedText` argument and use the first part as the function name and the second part as the event name.

## Links

- [Webinar on YouTube](https://www.youtube.com/watch?v=OUpUmWA0ae0)

- [Original article](https://www.moesif.com/blog/technical/serverless/debug-lambda-functions-locally-with-the-sam-cli-and-vscode/#)
- [Variable reference](https://code.visualstudio.com/docs/editor/variables-reference) for VSCode's launch.json

- [Kay's Twitter account](https://twitter.com/k4y1s)
- [Dashbird resources](https://dashbird.io/resources/) with serverless tutorials & case-studies.
- [Moesif blog](https://www.moesif.com/blog/) with API design tipps and tutorials.
