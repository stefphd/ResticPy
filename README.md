# ResticPy

A Python wrapper for [Restic](https://restic.net/) using a json configuration file.

Tested with Linux (ArchLinux) and Windows (version 10).

## Pre-requisites

For Linux and developing in Windows:

* `Restic` >= 0.13.0, e.g. from [here](https://github.com/restic/restic/releases). For details on Restic usage see e.g. [this](https://restic.readthedocs.io/en/latest/).
* `Python` >= 3.10.0
* `argparser` >= 1.4.0 Python package

For installation in Windows no pre-requisites are necessary (all binaries are provided with the installer).

You may install automatically the Python packages using

```bash
pip install -r requirements.txt
```

However, for ArchLinux users it is suggested to use pacman (requires root permission)

```bash
pacman python-argparse
```

## Installation

### Linux

For ArchLinux users use the AUR from [here](https://aur.archlinux.org/packages/resticpy).
For other distros run `install`. This just copies `resticpy` to `usr/local/bin` and checks for pre-requisites. Default installation is system-wide. For local installation use

```
./install --prefix $HOME/.local
```

Use `./uninstall` for uninstalling (with `--prefix` for local installation).

### Windows

Run the installer `ResticPy-X.Y-Windows10.msi`. Default installation directory is `C:\Program Files\stefphd\ResticPy\`. After the installation, it is necessary to add the installation directory to the PATH environment variable.

## Quick start

First, you need to generate the configuration file

```bash
resticpy genconf
```

which is placed in `$HOME` (`%USERPROFILE%` in Windows, i.e. `C:\Users\username`) with name `restic-conf.json`. You need to modify this file according to your requirements. See [Configuration](#configuration) for details on the configuration file.

Second, you can backup, forget older backup (if any) and print all snapshots using

```bash
resticpy -bfs
```

See [Usage](#usage) for details on the usage.

## Configuration

A `*.json` configuration file must be created for each repository. A file called `restic-conf.json` is used by default. The searching path for the configuration file are the following:

* `$HOME`
* `$HOME/.config`
* current directory (i.e. `./`)

A default configuration file can be generated using

```bash
resticpy genconf
```

with additional optional parameters (use `resticpy genconf --help` for the help):

* `--name NAME`: set configuration file name (defualt `restic-conf.json`)
* `--dir DIR`: set configuration file location (defualt `$HOME`)
* `--repo REPO`: set location of the repository (defualt is `$HOME/restic-backup`)
* `--passwd PASSWORD`: set repository password (defualt is `password`)
* `--init`: initialize the repository (see also `resticpy --init`)

Different or multiple configuration file(s) (e.g. one for each repository) or different searching folder(s) may be specified (see `resticpy --conf FILE(s)/FOLDER(s)`).

An example `*.json` file is the following.

```json
{
    "repo": "/path/to/backup",
    "passwd": "password",
    "dry-run": false,
    "init": false,
    "entry": [
        {
            "tag": ["home", "data"],
            "source": ["${HOME}"],
            "keeplast": "NaN",  
            "skip": false,
            "exclude": ["**/.*",
                        "**/*.o",
                        "**/*.o.d",
                        "**/.local"
                        ]
        },
        {
            "tag": ["config"],
            "source": ["/etc", "${HOME}/.config"],
            "sudo": true,
            "skip": false,
            "exclude": ["**/.*"]  
        }
    ],
}
```

Mandatory keys are:

* `repo`: repository directory (type string). A remote directory may be also used (e.g. with `sftp`)
* `passwd`: repository password (type string)
* `entry`: list of repository entries (array)
* `source`: source directories of the repository entry (array of strings)

Optional keys are (if missing then default values are used):

* `init`: `true` if the repositori has been initialized (default `false`)
* `dry-run`: force dry-run flag (default `false`)
* `tag`: tags of the repository entry (array of strings)
* `keeplast`: number of last snapshots to keep for the repository entry (integer). Use `"NaN"`, `"Inf"`, or -1 to keep all repositories (default is `-1`)
* `exclude`: excluded directories for the repository entry (array of stings)
* `sudo`: flag to run the restic command with sudo (default is `false`). Requires `sudo` installed. Working only for Linux (no effects in Windows)
* `skip`: flag to skip the operations on the entry (default is `false`).

Note that the keys `init` must be initially set to `false` (or neglected) and will be updated automatically by the software after the repository initialization.

## Usage

Print the help

```bash
resticpy --help
```

Initialize the repository

```bash
resticpy --init
```

Backup the repository

```bash
resticpy --backup
```

Print the list of snapshots

```bash
resticpy --snapshoots
```

Forget the snapshots according to the specified `keeplast` policy in the configuration file

```bash
resticpy --forget
```

Forget the snapshots according to the specified policy (see restic documentation for possible policy)

```bash
resticpy --forget POLICY
```

Mount a repository (only one configuration file allowed)

```bash
resticpy --mount path/to/mount
```

Specify either one or more (if allowed) configuration file(s) or searching folder(s) (but not both)

```bash
resticpy --conf path/to/jsonfile
```

Echo the restic command only

```bash
resticpy --backup --echo
```

Force dry-run in restic

```bash
resticpy --backup --dry-run
```

One-char (first char of the corrisponding command) and multiple arguments may be also used, e.g. to backup, forger, and print the snapshots

```bash
resticpy -bfs
```

or backup with dry-run

```bash
resticpy -bd
```

## Building in Windows

Windows (standalone) executable is built using `pyinstaller`. To reduce the generated file size, it is suggested to create a virtual environment using `venv`.

Before building, it is necessary to download Restic and put the executable `restic.exe` in the folder `resource` (see https://github.com/restic/restic/releases).

### Setup the virtual environment

To setup the virtual environment, first go to the source directory and run in the command prompt:

```bash
python -m venv env
```

where `env` is a local folder containing the virtual environment. To activate the create virtual environment run

```bash
cd env/Scripts
activate.bat
cd ../..
```

Finally, install the build requirements using

```bash
pip install -r requirements.txt
pip install -r build_requirements.txt
```

This actually installs `arg-parse` and `pyinstaller` in the created virtual environment. The `pip list` command should return the following output (package versions may differ)

```txt
Package                   Version
------------------------- ---------
altgraph                  0.17.2
future                    0.18.2
pefile                    2022.5.30
pip                       21.2.4
pyinstaller               5.1
pyinstaller-hooks-contrib 2022.7
pywin32-ctypes            0.2.0
setuptools                58.1.0
```

To exit from the virtual environment (after the building) use

```bash
cd env/Scripts
deactivate.bat
cd ../..
```

### Build the standalone application

To build the standalone application, use in the command prompt

```bash
python build
```

This creates a new folder `bin/resticpy` containing the binary files.

### Build the installer

An `.msi` installer can be built starting from the binary files in `bin/resticpy` using Visual Studio (>= 2019). See the Visual Studio project in `installerWin`. This may be done also from command prompt

```bash
cd \installerWin
"C:\Program Files (x86)\Microsoft Visual Studio\2019\Community\VC\Auxiliary\Build\vcvarsall.bat" x64
devenv installerWin.sln /build
```
