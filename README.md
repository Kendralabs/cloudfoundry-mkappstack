# cloudfoundry-mkappstack
simple automation to create a CF multi application stack

Description
===========

the main purpose is to store live CF spaces and redeploy
these later on, also to create a recipe for a whole CF
application stack to be automatically deployed into a
CF space some time later

it is possible to manipulate CF entitles, namely:
* applications
* service brokers
* services
* service instances
* user-provided service instances

the YAML setup is based on CF application manifests in 
most parts (applications list), whereas other lists are
proprietary for the automation

there are 6 important application keys, that need
to be considered while constructing the CF application stack:

* name - used to name the CF application, also used to construct artifact URL if necessary
* env::VERSION - used to decide whether to upgrade CF application with an artifact, also used to construct artifact URL if necessary
* env::artifact_name - (optional) used to construct artifact URL if an application name is not matching artifact name
* env::artifact_srcurl - (optional) used for artifacts that don't match general artifact URL scheme
* env::upsi_names - (optional) indicates the application is providing an user-provided service instance of given name
* env::service_broker_names - (optional) indicates the application is providing a service broker of given name

Usage
=====
the main program is a GNU Makefile, that uses certain
configuration files to execute targets - all described
below. configuration variables can be overriden using
command line make call, i.e.
```
make somevar=somevalue othervar=othervalue target
```

the artifacts need to be zip files:
* containing app manifest.yml (if absent, all required data needs to be a part of full stack manifest)
* containing all other app files, i.e.:
```
target/artifact-0.0.1.jar
manifest.yml
```
* files in zip archive should not be enclosed by any top directory structure

configuration files:
* secret.mk should contain CF setup, including confidential data
* appstack.mk contains all settings, including manifest file names
* manifest files (YAML format, as in *.yml.tmpl)

available make targets:
* cfset - download and set up CF CLI binary, authenticate and target CF space
* discover - retrieve targeted space information and create matching manifest file
* artifact-pack - retrieve all artifacts mentioned in manifest, put all into a tar.gz package
* deploy - install appstack in the CF org/space, according to the main manifest
* clean - wipe all local artifacts
* cfclean - wipe most of CF self-created objects (according to manifests)
* wipeall - cfclean + clean

Platform Deployment
=======

If you're following instructions from  [Platform Deployment Procedure: bosh deployment](https://github.com/trustedanalytics/platform-wiki/wiki/Platform-Deployment-Procedure:-bosh-deployment) take the  actions described below.

From this point there are two paths you can follow to deploy TAP. The path you will choose depends on the way how you obtained artifacts package.

**Solution 1** - Zip artifacts obtained from TAP team or prepared by building all components from sources.

* Copy templates to new files `appstack.mk` and `secret.mk`:
```
cp appstack.mk.tmpl appstack.mk
cp secret.mk.tmpl secret.mk
```
* Enter your environment information to `secret.mk`. Edit cloudfoundry api endpoint, user, password, org & space.
* Set path to the artifacts.
* Open `appstack.mk` file and set artifact_pfx: `artifact_pfx = file://<artifacts_directory>`.

_Note: if your artifacts are stored in `/tmp/PACKAGES` directory, your artifact_pfx should be set to: `artifact_pfx = file:///tmp/PACKAGES` (remember about "file://" prefix!)_

Fields afcturl and stack_mflist depend on whether your zipped artifact file names contain versions or not.
Please, check the names format of zipped artifacts in artifacts directory.

**If they do contain versions** and are in the following format: `<appname>-<version>.zip` (for example: `app-launcher-helper-0.4.5.zip`) take the following steps:
  * in appstack.mk set the following afcturl: `afcturl = $(artifact_pfx)/$(appname)-$(appver).zip`
  * in appstack.mk set the following stack_mflist: `stack_mflist = versions.yml settings.yml appstack.yml`

**If they do not contain versions** and are in the following format: `<appname>.zip` (For example: `app-launcher-helper.zip`)
  * in appstack.mk set following afcturl: `afcturl = $(artifact_pfx)/$(appname).zip`
  * in appstack.mk set the following stack_mflist: `stack_mflist = settings.yml appstack.yml`


**Solution 2** - Zip artifacts obtained from artifacts repository (internal or public)

**_Note: This solution is under development and is not available yet_**

Continue with [Platform Deployment Procedure: bosh deployment](https://github.com/trustedanalytics/platform-wiki/wiki/Platform-Deployment-Procedure:-bosh-deployment)
