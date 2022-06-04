# ResticPy

A Python wrapper for [Restic](https://restic.net/) using a json configuration file.

Tested with Linux (ArchLinux) and Windows (version 10).

## Pre-requisites

* `Python` >= 3.10.0
* `Restic` >= 0.13.0, e.g. from [here](https://github.com/restic/restic/releases). For details on Restic usage see e.g. [this](https://restic.readthedocs.io/en/latest/).
* `argparser` >= 1.4.0 Python package

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
For other distros just copy `resticpy` to `/bin/usr` (or other directories in the `$PATH`).

### Windows

For now no installation is possible. A msi package with an exe will be probably created.

## Configuration

A `*.json` configuration file must be created for each repository. 
A file called `restic-conf.json` is used by default. The searching path for the configuration file are the following:

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

## Basic usage

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

Forget the snapshots according to the specified `keeplast`

```bash
resticpy --forget
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
