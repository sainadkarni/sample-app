# SampleApp

This project was developed to try and learn GitHub actions to deploy a SPA.

### Creation of GitHub Actions file

First off, create a directory `.github/workflows/` and create a YML file which will define the jobs to be run for that specific action.
The actions file starts out with,

```
name: workflow name
```

### Workflow trigger

Assign a trigger to start off your workflow. This can be a push to a specific branch, a PR acceptance etc.

```
on:
  push:
	branches:
	  - 'master'
```
[Full guide to events that can trigger workflows](https://docs.github.com/en/actions/learn-github-actions/events-that-trigger-workflows)

### Job specific instructions (Angular here)

This section details the job specific instructions for the YML file. Since we are trying to deploy an Angular application here, the key things to note are to install NPM on the job runner to be able to build the application.

```
jobs:
  build:
	name: Build
	runs-on: ubuntu-latest
	steps: <<steps will follow here>>
```

The default Ubuntu hosted GitHub job runner will be used with this configuration.
+ [GitHub Job Runner](https://docs.github.com/en/actions/using-github-hosted-runners/about-github-hosted-runners)
+ [Custom (Self hosted) Job Runner](https://docs.github.com/en/actions/hosting-your-own-runners/about-self-hosted-runners)

### Check out source code

The defined job does not pull source code by default, you need to specify a source. This step utilizes a default GitHub action called `checkout` which does just that.

```
- name: Checkout
  uses: actions/checkout@v1
```

### Install NodeJS

Configure the runner with NodeJS as we need NPM

```
- name: Use Node 12.x
  uses: actions/setup-node@v1
  with:
    node-version: '12.x'
```
[Detailed instruction about various runtimes available](https://docs.github.com/en/actions/automating-builds-and-tests/about-continuous-integration)

### Install dependencies and run build command

This is the Angular specific part of the documentation, which first runs `npm ci` to install any dependencies and then runs the build command according to the `package.json` file which basically runs `ng build` with the specified options from the `angular.json` file.

```
- name: Install dependencies
  run: npm ci
- name: Build
  run: npm run build
```

### Upload artifacts

The build artifacts must be placed in an appropriate directory for the next deploy job to be able to pick them up. Here, the build artifact will be the generated `dist` folder from the previous run of `ng build`.

```
- name: Archive build
  if: success()
  uses: actions/upload-artifact@v1
  with:
    name: deploy_dist
	path: dist
```

`if: success()` is used to run this step only if the previous step (build) passed.