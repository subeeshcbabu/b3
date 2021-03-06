Buildorch
=========

[![NPM version](https://badge.fury.io/js/buildorch.svg)](http://badge.fury.io/js/buildorch)
[![Build Status](https://travis-ci.org/subeeshcbabu/buildorch.svg?branch=master)](https://travis-ci.org/subeeshcbabu/buildorch)


[![NPM](https://nodei.co/npm/buildorch.png)](https://nodei.co/npm/buildorch/)

# Overview
A Simple CLI based utility to Orchestrate NodeJs Application builds.
The workflow has these three,
- Build  (NPM Install, Bower Insall)
- Bake   (Execute the Grunt tasks - lint,test,cover,build OR execute the npm scripts lint,test,cover,build)
- Bundle (Bundle the Application code and node_modules - Default format is tgz)

## Usage

Init script to install nodejs and npm and configure the npm defaults
	
	curl -k https://raw.githubusercontent.com/subeeshcbabu/buildorch/master/buildorch.sh -o buildorch.sh
    . buildorch.sh
    installNode

Install the module
	
	npm install buildorch

Commands
	
	node_modules/.bin/buildorch b3  (Executes build, bake and bundle)
	node_modules/.bin/buildorch b2  (Executes build and bake)
	node_modules/.bin/buildorch build
	node_modules/.bin/buildorch bake
	node_modules/.bin/buildorch bundle
	
### Pre Requisites
Posix OS

The commands should be executed at application root 

## Features:

### Configuration based 

The build orchestartion works based on JSON Configuration `buildorch.json`. If the application did not specify a config json file, a default template is loaded [buildorch.json](https://raw.github.com/subeeshcbabu/buildorch/master/config/buildorch.json). The config loader supports arbitrary types using the module [shortstop] (https://github.com/krakenjs/shortstop).

A Sample `buildorch.json`
```js

{
	"init" : {
		"clean"	: [
			"path:node_modules"
		],
		"script" : ""
	},

	"build" : {
		"preclean"	: [
			
		],
		"files" : [
			"getit:https://raw.githubusercontent.com/subeeshcbabu/buildorch/master/buildorch.sh"
		],
		"prescript" : "path:prebuildscript.sh",
		"execbuild" : {
			"npm" : {
				"command" : "npm",
				"task" : "install",
				"skip" : "exec:./node_modules/buildorch/lib/util/validateutil#skipNpmInstall",
				"failonerror" : true
			},
			"bower" : {
				"command" : "path:node_modules/bower/bin/bower",
				"task" : "install",
				"skip" : "exec:./node_modules/buildorch/lib/util/validateutil#skipBowerInstall",
				"failonerror" : false
			}
		},
		"postscript" : "path:postbuildscript.sh",
		"postclean"	: [
			
		]
	},

	"bake" : {
		"files" : [

		],
		"scriptprefix" : "/bin/bash", // OR "/usr/local/node/bin/node" - Optional
		"prescript" : "path:prebakescript.sh", 
		"execbake" : {
			"lint" : {
				"task" : "lint",
				"skip" : "env:SKIP_LINT|b",
				"failonerror" : true
			},
			"unittest" : {
				"task" : "test",
				"skip" : "env:SKIP_TEST|b",
				"failonerror" : true
			},
			"coverage" : {
				"task" : "coverage",
				"skip" : "env:SKIP_COVERAGE|b",
				"failonerror" : true
			}
		},
		"postscript" : "path:postbakescript.sh",
		"postclean"	: [
			
		]
	},

	"bundle" : {	
 		"files" : [

		],	
		"prescript" : "path:prebundlescript.sh",
		"execbundle" : {
			"target" : "target",
			"format" : "tar"
		},
		"postscript" : "path:postbundlescript.sh",
		"postclean"	: [
			
		]
	},
	"metrics" : {
		"write" : {
			"outfile" : "path:build-metrics.json"
		}
	}
}
```
#### Handlers
##### [shortstop-handlers] (https://github.com/krakenjs/shortstop-handlers) - path, file, env, exec etc.

##### getit handler

format - `getit:<url>#[target_location]#[ENV shouldDownload?],[nooverride]`

- <url> - The remote URL from where the content is downloaded.

- [target_location] - optional. The local path (relative to cwd) to save the file.

- [ENV shouldDownload?] -  optional. The ENV to decide whether to download the file or not. `process.env.[ENV  shouldDownload?]` should be true to download the file.

- [nooverride] - optional. If added, the download will not happen if file exists.

This handler will download the file and save it to the `process.cwd()/<filepath>` location.

The scripts and other executers can reference the file using `process.cwd()/<filepath>` path.

Eg:- 

getit:https://raw.githubusercontent.com/subeeshcbabu/buildorch/master/buildorch.sh#build/build_init.sh

getit:https://raw.githubusercontent.com/subeeshcbabu/buildorch/master/buildorch.sh#build_init.sh#REQUIRE_BUILD,nooverride

getit:https://raw.githubusercontent.com/subeeshcbabu/buildorch/master/buildorch.sh#abc.sh#nooverride


### Tasks

##### clean

This can be `clean`, `preclean` or `postsclean`
Clean the list of files/directories. This task can be added as a sub task for any Main tasks (init, build, bake, bundle etc)
```javascript
	"clean"	: [
		"path:node_modules"
	]
```
##### script

This can be `script`, `prescript` or `postscript`.

Execute any script by specifying the relative, absolute or remote file location. 
```javascript
	
	"script" : " path:nodefile.js"

	"script" : "getit:https://raw.githubusercontent.com/subeeshcbabu/buildorch/master/buildorch.sh#build/build_init.sh"

```

##### files

Download config/script files from remote location
```javascript
	"files"	: [
		"getit:https://raw.githubusercontent.com/subeeshcbabu/buildorch/master/buildorch.sh#build/build_init.sh"
	]
```
##### init

Gets executed automatically before command execs (b3, b2, build etc). You can add any one of the sub tasks for this (clean,files, script)

```javascript
		"init" : {
			"clean"	: [
				"path:node_modules"
			],
			"script" : ""
			"files"	: [
				"getit:https://raw.githubusercontent.com/subeeshcbabu/buildorch/master/buildorch.sh#build/build_init.sh"
			]
		}
```
##### build

To execute the build task. By Default `npm install` and `node_modules/bower/bin/bower install` are executed.
The default validator used for npm install and bower install checks are defined in `lib/util/validateutil`

```javascript
	"build" : {
		"files" : [

		],
		"script" : "",
		"execbuild" : {
			"npm" : {
				"command" : "npm",
				"task" : "install",
				"skip" : "exec:./node_modules/buildorch/lib/util/validateutil#skipNpmInstall",
				"failonerror" : true
			},
			"bower" : {
				"command" : "path:node_modules/bower/bin/bower",
				"task" : "install",
				"skip" : "exec:./node_modules/buildorch/lib/util/validateutil#skipBowerInstall",
				"failonerror" : false
			}
		},
		"clean"	: [
			
		]
	}
```
##### bake

To execute the tasks - `lint`, `unittest`, and `coverage`. The default task runner is [Grunt](http://gruntjs.com/).

The `command` can be used to specify the tool/module used to execute the task. Another example would be `npm run-script`.
It will execute the commands in sequential order. 

Also, you can define a custom task using the `custom` key.
The SKIP_LINT, SKIP_TEST and SKIP_COVERAGE are already defined part the default config. Developers are free to use any ENV property for this using the [Shortstop env handler](https://github.com/subeeshcbabu/shortstop-handlers#handlersenv)

The `failonerror` is by default set to `true`.

npm run-script lint
npm run-script test
npm run-script coverage
npm run-script build

```javascript
	"bake" : {
		"files" : [

		],
		"scriptprefix" : "",
		"script" : "",
		"execbake" : {
			"lint" : {
				"task" : "lint",
				"skip" : "env:SKIP_LINT|b",
				"failonerror" : true
			},
			"unittest" : {
				"task" : "test",
				"skip" : "env:SKIP_TEST|b",
				"failonerror" : true
			},
			"coverage" : {
				"task" : "coverage",
				"skip" : "env:SKIP_COVERAGE|b",
				"failonerror" : true
			},
			"custom" : {
				"task" : "build",
				"skip" : "env:SKIP_CUSTOM|b",
				"failonerror" : true
			}
		},
		"clean"	: [
			
		]
	}
```
##### bundle

To bundle/assemble the source files to a predefined format.
```javascript
	"bundle" : {	
 		"files" : [

		],	
		"script" : "",
		"execbundle" : {
			"source" : "path:source",
			"target" : "path:target",
			"format" : "tgz", OR (tar, zip, copy, custom)
			"ignorefile" : [
				"path:.packageignore"
			]
		},
		"clean"	: [
			
		]
	}
```

`source` The source directory to copy over the files after executing the exclude list/ignore patterns. 

`ignorefile` The list of files o specify the ignore patterns.

By default the following files are ignored [Default patterns] (https://raw.githubusercontent.com/subeeshcbabu/buildorch/master/config/.defaultignore). Plus the `.packageignore` is automatically loaded if its present.


Add a .packageignore file (Make sure the file extension is not .txt) in the application root directory, to add the list of files/directories to be ignored. This supports regex expressions to match/find the list of files and works exactly like the `.gitignore`, `.jshintignore` etc

`target` The target directory to save the bundled file. Default is process.cwd()/'target'.

`format` The bundle format. Supported formats are `tar`, `tgz` and `zip`.


In addition to this there are two special types of values for this parameter.

`copy` This copies over the files from the process.cwd() to `source`. User can implement their own custom bundle process and invoke it using `script`.

`custom` This completely ignores the bundle steps and uses whatever custom implement user specifies using `script`


##### metrics

To generate the `build-metrics.json` file. 
```javascript
"metrics" : {
		
	"write" : {
		"outfile" : "path:build-metrics.json"
	}
}
```

### Generates build metrics

The orachestartor generates a `build-metrics.json` file, helpful in figuring out the time spent on granular level tasks and status of individual steps.

A sample `build-metrics.json`

```js
{
  "application" : "foo",
  "userid" : "bar",
  "machine" : "blah",
  "action" "build",
  "nodejsVersion": "v0.10.25",
  "builderVersion": "0.0.7",
  "starttime": "2014-04-29 17:44:47",
  "init": {
    "startTime": "2014-04-29 17:44:47",
    "endtime": "2014-04-29 17:44:49",
    "status": "SUCCESS"
  },
  "build": {
    "startTime": "2014-04-29 17:44:49",
    "npm": {
      "starttime": "2014-04-29 17:44:59",
      "endtime": "2014-04-29 17:45:24",
      "status": "SUCCESS"
    },
    "gruntcli": {
      "starttime": "2014-04-29 17:45:24",
      "endtime": "2014-04-29 17:45:28",
      "status": "SUCCESS"
    },
    "endtime": "2014-04-29 17:45:28",
    "status": "SUCCESS"
  },
  "bake": {
    "startTime": "2014-04-29 17:45:28",
    "lint": {
      "starttime": "2014-04-29 17:45:28",
      "endtime": "2014-04-29 17:45:28",
      "status": "SUCCESS"
    },
    "unittest": {
      "starttime": "2014-04-29 17:45:28",
      "endtime": "2014-04-29 17:45:29",
      "status": "SUCCESS"
    },
    "coverage": {
      "starttime": "2014-04-29 17:45:29",
      "endtime": "2014-04-29 17:45:35",
      "status": "SUCCESS"
    },
    "custom": {
      "starttime": "2014-04-29 17:45:35",
      "endtime": "2014-04-29 17:45:38",
      "status": "SUCCESS"
    },
    "endtime": "2014-04-29 17:45:38",
    "status": "SUCCESS"
  },
  "bundle": {
    "startTime": "2014-04-29 17:45:38",
    "endtime": "2014-04-29 17:45:45",
    "status": "FAILURE"
  },
  "endtime": "2014-04-29 17:45:45",
  "status": "FAILURE"
}
```

### TODO

- actual time spent in addition to starttime and endtime
- number test failures, number of lint errors
- Error codes


