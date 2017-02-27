# ESDeploy - An Elastic Search schema deployment command line tool

## Overview
ESDeploy is an autotmated schema (change management) tool to help you version your Elastic Search backend. Its a simple tool that 
follows convention over configuration. The goal is to treat your schema changes like code, so for each change needed there is a schema file
that represents the change. Best practices suggest those files should be checked into source control allowing for repeatable versioning of your
Elastic Search backend across all environments (development, staging, production)

## Conventions
- Scripts are only applied once and never run a second time.
- Scripts are run in the order they are represented on disk (sorted alphabetically).
- Only *.js files are executed.
- The unique identifier for a script is the folder and file name so don't renamme folders or files.
- Scripts that are executed successfully are logged into an index called esdeploy_v1 (alias = esdeploy)
- Currently there is one mapping inside the esdeploy index called version_info

## Getting Started 
1. Create a folder to store your Elastic Search schema changes
1. Name the file with extension js (*.js) and its recommended to use a numbering scheme to keep things sorted correctly
1. Download esdeploy which is a single executable
1. esdeploy --help will list out all available options
    - DryRun will verify what scripts will be run against elastic search without making any changes
    - Deploy will apply the scripts to elastic search

```
$ esdeploy --help
usage: deploy [<flags>] <command> [<args> ...]

A command-line deployment tool to version Elastic Search.

Flags:
  --help  Show context-sensitive help (also try --help-long and --help-man).

Commands:
  help [<command>...]
    Show help.

  dryrun [<flags>] <url>
    Performs a dry run listing out changes that would be made

  deploy [<flags>] <url>
    Deploy elastic search changes

```

## JS File Standard
- First line is HTTP verb (POST, PUT, DELETE, HEAD)
- Second line is the partial URL to elastic resource (See example below)
- Rest of file contains JSON used to make schema change

## Examples

Assuming we have the blow folder structure. This folder is only used to logically group related scripts. In this example
we are dealing with a "cars" and "boats" elastic search index.

### Folder structure
```
-- cars
----- 01.001_create_cars_index.js
----- 01.002_create_bmw_mapping.js
----- 01.002_create_cars_alias.js

-- boats
----- 01.001_create_boat_index.js
----- 01.002_create_speedboat_mapping.js
```

If we examine the contents of the 01.001_create_cars_index.js file we Getting
```
POST
cars/
{
	"settings" : {
		"index" : {
			"number_of_shards" : 5,
			"number_of_replicas" : 1,
			"mapper": {
				"dynamic":false
			}       
		}
	}
}
```

Notice the following
- First line tells us this is a POST
- Second line tells us this is at cars/ url. (Fully qualified assuming localhost this would be http://localhost:9200/cars/)
- Third line - EOF is the json payload to create the new index.

The nice thing about this format is that as you test your elastic search index creation using Postman or similar tools the same JSON content 
can then be used for this script without alteration.

## Command Line Details

## dryrun
Will perform a dry run first validating your scripts and informing you of what scripts will be applied

### Parameters
- url: Url of Elastic Search server. Should be in format of http://[server]:port. (Required)

- folder: Location of scripts. If not supplied then the current executing folder will be used (Optional)

```
$ esdeploy dryrun --help
usage: esdeploy dryrun [<flags>] <url>

Performs a dry run listing out changes that would be made

Flags:
      --help        Show context-sensitive help (also try --help-long and
                    --help-man).
  -f, --folder="."  Folder containing schema js files

Args:
  <url>  Elastic Search URL to run against

Example:
--------

esdeploy dryrun http://localhost:9200 -f ./escripts

```

## deploy
Will deploy your scripts to ElasticSearch and list out changes applied

### Parameters
- url: Url of Elastic Search server. Should be in format of http://[server]:port. (Required)

- folder (f): Location of scripts. If not supplied then the current executing folder will be used (Optional)

- silent (s): Don't prompt for confirmation of deploying changes.



```
$ esdeploy deploy --help
usage: esdeploy deploy [<flags>] <url>

Deploy elastic search changes

Flags:
      --help        Show context-sensitive help (also try --help-long and
                    --help-man).
  -f, --folder="."  Folder containing schema js files
  -s, --silent      Don't prompt for confirmation, run silently

Args:
  <url>  Elastic Search URL to run against

Example:
--------

esdeploy deploy http://localhost:9200 -f ./escripts -s

```

