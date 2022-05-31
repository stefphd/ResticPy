# ResticPy

A Python wrapper for [Restic](https://restic.net/) using a json configuration file.

Tested OS are Linux (ArchLinux) and Windows (version 10).

## Pre-requisites

* `Python` >= 3.10.0
* `Restic` >= 0.13.0, e.g. from [here](https://github.com/restic/restic/releases). For details on Restic usage see e.g. [this](https://restic.readthedocs.io/en/latest/).
* `argparser` >= 1.4.0 Python package

You may install automatically the Python packages using

```bash
pip install -r requirements.txt
```

## Configuration

A `*.json` configuration file must be created for each repository. An example `*.json` file is the following.

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
* `sudo`: flag to run the restic command with sudo (default si `false`). Requires `sudo` installed. Working only for Linux (no effects in Windows)

Note that the keys `init` must be initially set to `false` (or neglected) and will be updated automatically by the software after the repository initialization. The software will also add a new key `jsonfile` automatically for internal usage.

A configuration file called `conf.json` is used by default. Different or multiple configuration files may be specified (e.g. one for each repository).

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

Specify one or more (if allowed) configuration files

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
