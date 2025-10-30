# Setting up an Apptainer for Pynguin Development

## Overview

The container is an [Apptainer](https://apptainer.org/) sandbox, which is a read-write container within a directory structure. The container is a writable directory, in which programs can be installed after building the container. 

The container hosts the [Pynguin](https://github.com/se2p/pynguin) source code and installation, and the Python environment to run Pynguin. The host system provides a directory, which is mounted to the container. This directory contains the Python modules under test and Pynguin writes the test results and reports to it.

## Instructions

 1. Clone this repository on a host system where the Apptainer should run and where Apptainer is installed. At least, copy the definition file `pynguin-dev-apptainer.def` to the host system.

 2. Build the Apptainer 
 
    The Apptainer is defined by `pynguin-dev-apptainer.def`, which has several parameters whose values can be changed if needed:
    
    1. `PYTHON_VERSION=3.10`: Pynguin currently requires Python 3.10.
    2. `PYNGUIN_GIT_URL=https://github.com/se2p/pynguin.git`: The URL to Pynguin's git repository. Change it if you work on a fork of Pynguin.
    3. `PYNGUIN_DEV=/home/pynguin-dev`: The directory in the Apptainer where Pynguin will be cloned and installed.
    4. `PYNGUIN_DATA=/mnt/pynguin-data`: The mounting point for a folder of the host system that stores the modules under test and that stores the test results and reports.
    5. `VIRTUALENVS_HOME={{PYNGUIN_DEV}}/virtualenvs`: The directory in the Apptainer where the Python virtual environment will be installed.

    Additionally, when building the Apptainer, further tools can be installed such as `nano`. Have a look at the definition file and add the tools to install via `apt-get`.

    **Build the Apptainer**

    In the host system's directory containing the definition file: 

    `apptainer build --sandbox pynguin-dev-apptainer pynguin-dev-apptainer.def`

    This command takes a while and builds the container named `pynguin-dev-apptainer` (a different name can be used). The container is the newly created folder with the same name. 

    When building the container, `poetry` will be installed and Pynguin will be cloned and installed (see the definition file).

3. Start the Apptainer

    `apptainer shell --writable --no-home --bind <data directory in the host system>/:/mnt/pynguin-data pynguin-dev-apptainer`

    where: 
    * `shell` opens a shell to the started container.
    * `--writable` makes the container writable/mutable. 
    * `--no-home` ensures that the home directory of the host system is not mounted to the container. 
    * `--bind` mounts the data directory in the host system to `/mnt/pynguin-data` in the container (see parameter `PYNGUIN_DATA`). 
    * `<data directory in the host system>` is the host system's folder, which contains the modules under test and to which Pynguin writes the test outputs (test cases) and test reports.
    * `pynguin-dev-apptainer` is the container (directory name) to be started.

4. Run Pynguin in the Apptainer

    Inside the apptainer, go to the `PYNGUIN_DEV` folder and run the following command:

    `./virtualenvs/pynguin-OLitu6Ry-py3.10/bin/pynguin --project-path /mnt/pynguin-data/subjects/ --output-path /mnt/pynguin-data/results/output/ --report_dir=/mnt/pynguin-data/results/reports/ --module-name example -v`

    where:
    * `./virtualenvs/pynguin-OLitu6Ry-py3.10/bin/pynguin` is the path to the Python virtual environment where Pynguin is installed. The folder/environment `pynguin-OLitu6Ry-py3.10` will have a different name in your case.
    * `--project-path` names the folder containing the project (one or more Python modules) to be tested.
    * `--output-path` names the folder to which Pynguin writes the test results (test cases).
    * `--report_dir` names the folder to which Pynguin writes the test reports.
    * `--module-name` names the Python module (file) for which tests should be generated.
    * `-v` enables a more verbose output in the shell.

    For this command, we copied before the module `example.py` to `mnt/pynguin-data/subjects/` (see [Pynguin Quickstart](https://pynguin.readthedocs.io/latest/user/quickstart.html)).


## Remote development with VS Code

Assuming that the Pynguin Apptainer runs on a server and that an SSH connection can be established to the server, we can use [VS Code's Remote Development using SSH](https://code.visualstudio.com/docs/remote/ssh) to connect to the server and the Apptainer to work on the Pynguin code within the Apptainer and execute Pynguin on the server.

For this purpose, the VS Code client requires at least the "Remote-SSH" extension. The "Remote Development" extension pack comprises several extensions for remote development.

On the machine running VS Code, add the following to `~/.ssh/config`:

```
Host pynguin-dev-apptainer~*
  RemoteCommand apptainer shell --writable --no-home --bind <data directory in the host system>/:/mnt/pynguin-data <fully qualified path name to the folder containing the Apptainer>/pynguin-dev-apptainer/
  RequestTTY yes

Host <server-name> pynguin-dev-apptainer~<server-name>
  HostName <host-name of the server>
  User <user-name> 
```

Fill in the placeholders (`<...>`). The remote command is the same as before to start the Apptainer. The only difference is that the fully qualified name of the Apptainer is used. (Source: [Remote container with Singularity](https://github.com/microsoft/vscode-remote-release/issues/3066#issuecomment-1019500216) -- Apptainer is the successor of Singularity)

In VS Code, open "a remote window" or "connect current window to host". Select the connection `<server-name> pynguin-dev-apptainer~<server-name>` (see `~/.ssh/config` above) and authenticate.

VS Code is now connected to the running Apptainer on the server. To program Pynguin directly inside the Apptainer:

* Open the folder `PYNGUIN_DEV/pynguin`. The Pynguin source code located in the Apptainer is now editable from VS Code.
* You might have to install the Python extensions.
* Select the Python Interpreter by entering the path to `python` or `pynguin` in the virtual environment created in the Apptainer, e.g., `VIRTUALENVS_HOME/pynguin-OLitu6Ry-py3.10/bin/pynguin` (in general, the folder `pynguin-OLitu6Ry-py3.10` will likely have a different name). Now all dependencies of Pynguin code are resolved in VS Code, and Python or Pynguin can be triggered from VS Code and directly executed in the Apptainer.
* When opening a terminal in VS Code, the shell is directly connected to the Apptainer and Pynguin can be executed from this shell. 



