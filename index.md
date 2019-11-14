# JARVICE Application Definition Guide

The JARVICE Application Definition (Appdef) Reference describes the application interface for applications that are deployed on JARVICE with PushToCompute. This guide will walk through creating an AppDef for common application use cases on the JARVICE platform. 

Refer to this [link](https://jarvice.readthedocs.io/en/latest/appdef/) for complete documentation.

## Contents

* [Using AppDef to Customize Applications in JARVICE](#using-appdef-to-customize-applications-in-jarvice)
* [Default AppDef Template](#default-appdef-template)
* [Shell AppDef Template](#shell-appdef-template)
* [Batch AppDef Template](#batch-appdef-template)
* [Web service AppDef Template](#web-service-appdef-template)
* [Jupyter Notebook AppDef Template](#jupyter-notebook-appdef-template)


## Using AppDef to Customize Applications in JARVICE

All applications in JARVICE are created with the PushToCompute flow. PushToCompute is a full continuous integration/continuous deployment (CI/CD) pipeline for applications on the JARVICE platform. See [here](https://jarvice.readthedocs.io/en/latest/cicd/) for a complete overview.

Adding an AppDef to an applications enables the owner to customize a variety of components including: the application card, available workflows, workflow arguments, and connection options.

All new JARVICE applications will use the default AppDef template until it is replaced by the application owner. There are two options available to update an AppDef:

1) Add the AppDef to the application's container

The following snippet can be added to the end of an application's Dockerfile:

```bash
COPY AppDef.json /etc/NAE/AppDef.json
# Verify AppDef is valid
RUN curl --fail -X POST -d @/etc/NAE/AppDef.json https://api.jarvice.com/jarvice/validate
```

The application on JARVICE will be updated after a PushToCompute Pull or Build+Pull request is complete.

2) Upload an AppDef from the PushToCompute editor

* Open the editor by clicking the stacked menu in the upper left corner

![Application Card](screenshots/appCard.png)

* Select `APPDEF` option at the top right and click `LOAD FROM FILE` button 

![Edit AppDef](screenshots/editAppDef.png)

## Default AppDef Template

All new applications will use the default AppDef template. This section will examine the default template

Each AppDef contains the following sections:

#### name

Name that appears on the application icon

#### description

Description that appears on the application card

#### author

Author listed on the application icon

#### licensed

Is a license included with the application? `false` indicated an additional license is required

#### classification

Categories available on the JARVICE platform to filter applications

![Application Filter](screenshots/appFilter.png)

E.g. Software Development, Artificial Intelligence, Machine Learning, etc

#### machines

List the eligible machines an application can use.

[Machines](https://status.jarvice.com/) available on the NIMBIX cloud

#### vault-type

A Vault is a user's persistent storage mounted on `/data`. An application can restrict which vault types can be used

#### image

Base64 image data of icon to use for an application card.

```bash
cat image.png | base64 -w0
```

#### commands

A command describes a workflow provided by an application. The default template provides `Batch`, `Server`, and `GUI` options. 

### Sample Application

The different work flows are illustrated by `nimbix/ubuntu-desktop:xenial` example available on DockerHub. Create a new PushToCompute application:

1. Select PushToCompute tab on the menu on the right
2. Click the `New` icon in the middle of the page
3. Fill in `App ID` (e.g. `default`)
4. Fill in 'Docker or Singularity Repository' with `nimbix/ubuntu-desktop:xenial`
5. Click the `OK` button in the lower right corner
6. Click on the application menu and select Pull

![Application Card](screenshots/appCard.png)
![Pull](screenshots/pull.png)

The application icon will update when the Pull completes

![Ubuntu icon](screenshots/ubuntuApp.png)

The AppDef for `nimbix/ubuntu-desktop:xenial` sets the `name`, `author`, and `image` keys shown above. The additional components on the icon are the `App ID` (bottom right corner) and permissions setting (lock in top right corner). All new applications on JARVICE are set to private and will appear with the lock icon. Application can be made public following [these](https://jarvice.readthedocs.io/en/latest/public/) guidelines.

Clicking on the application icon will show the available workflows (Batch, Server, GUI)

![Workflows](screenshots/ubuntuWorkflow.png)

**Batch**

Starts a non-interactive job that exits after specified `Command` is complete.

![Batch Example](screenshots/batchExample.png)

The output log is shown when the job exits:

![Batch Run](screenshots/batchRun.png)

**Server**

Starts an interactive job users can connect to by clicking the screenshot below `Password`

![Server Example](screenshots/serverExample.png)

For ubuntu-desktop, this will open a vnc session in the web browser with a Linux desktop environment.

Note: users will need to shutdown the job using either the `Log Out` menu inside the application or clicking the red power button at the top right of a running job.


**GUI**

Start an interactive job that exits when the window for `Command` is closed

![GUI Example](screenshots/guiExample.png)

The job will exit if the running window is closed (see below)

![GUI Run](screenshots/guiRun.png)

The `GUI` option above require the Nimbix Desktop. This environment is installed by adding [image-common](https://github.com/nimbix/image-common#ubuntu-with-nimbix-desktop) to an applications Dockerfile.

## Shell AppDef Template

The `shell.json` AppDef is for applications that do not require a desktop environment. The JARVICE platform will connect interactive jobs to a [gotty](https://github.com/yudai/gotty) terminal if a desktop environment can not be found. This enables native support for applications pulled from a container registry such as DockerHub.

### Sample Application

Create a JARVICE application using the official CentOS image from DockerHub:

1. Select PushToCompute tab on the menu on the right
2. Click the `New` icon in the middle of the page
3. Fill in `App ID` (e.g. `shell`)
4. Fill in 'Docker or Singularity Repository' with `centos:7`

    ![Centos](screenshots/centosExample.png)

5. Upload `shell.json` using the `APPDEF` tab
6. Click the `OK` button in the lower right corner
7. Click on the application menu and select Pull

![Application Card](screenshots/appCard.png)
![Pull](screenshots/pull.png)

Submit a new job after the Pull completes:

1. Click on the application icon
2. Click the `Shell` button
    ![Shell Workflows](screenshots/shellWorkflow.png)
3. Click the `SUBMIT` button
    ![Shell Submit](screenshots/shellSubmit.png)
4. Connect to a running job by clicking `Click here to connect`
    ![Shell Connect](screenshots/shellConnect.png)


![Centos Shell](screenshots/centosRun.png)

Note: 

* sudo is not installed in the official DockerHub containers. The root password can be retrieved using `click to show` next to `Password` on the running job. Login to root using the `su` command.
* `shell.json` set the `machines` key to `n0` which will limits the available machine types for this application. Replace with `*` to support all machine types or add additional machine types to the list. [Available machine types on the NIMBIX cloud](https://status.jarvice.com/)

## Batch AppDef Template

The `batch.json` AppDef creates a non-interactive job by setting `interactive` to `false`. Users will be unable to connect to the running job. The job will exit when the `path` command exits.

```json
        "Batch": {
            "path": "/bin/bash",
            "interactive": false,
            "name": "Batch",
            "description": "Hello World",
            "parameters": {
                "-l": {
                    "name": "-l",
                    "description": "-l",
                    "type": "CONST",
                    "value": "-l",
                    "positional": true,
                    "required": true
                },
                "-c": {
                    "name": "-c",
                    "description": "-c",
                    "type": "CONST",
                    "value": "-c",
                    "positional": true,
                    "required": true
                },
                "command": {
                    "name": "Command",
                    "description": "Command to run",
                    "type": "STR",
                    "value": "echo Hello World",
                    "positional": true,
                    "required": true
                }
            }
        }
```

The above example uses `parameters` to pass additional arguments to the `path` command. This example will run the `command` given by the user in a bash login shell. The default command passed to the server will be `/bin/bash -l -c echo Hello World`.

Refer to the commands object, parameter object, and parameter type references [here](https://jarvice.readthedocs.io/en/latest/appdef/) for additional information.

### Sample Application

Create a JARVICE application using the official Ubuntu image from DockerHub:

1. Select PushToCompute tab on the menu on the right
2. Click the `New` icon in the middle of the page
3. Fill in `App ID` (e.g. `batch`)
4. Fill in 'Docker or Singularity Repository' with `ubuntu:xenial`

    ![Ubuntu](screenshots/ubuntuExample.png)

5. Upload `batch.json` using the `APPDEF` tab
6. Click the `OK` button in the lower right corner
7. Click on the application menu and select Pull

![Application Card](screenshots/appCard.png)
![Pull](screenshots/pull.png)

Submit a new job after the Pull completes:

1. Click on the application icon
2. Click the `Batch` button
3. Click the `SUBMIT` button
    ![Batch Submit](screenshots/ubuntuBatch.png)

![Ubuntu Batch](screenshots/ubuntuRun.png)

Note: 

* `batch.json` set the `machines` key to `n0` which will limits the available machine types for this application. Replace with `*` to support all machine types or add additional machine types to the list. [Available machine types on the NIMBIX cloud](https://status.jarvice.com/)
* `command` parameter is `STR` type with a default value of `echo Hello World`. This `Parameter` can be set by a user in step 3 above



## Web service AppDef Template

## Jupyter Notebook AppDef Template
