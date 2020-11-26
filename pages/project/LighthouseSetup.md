# ⛑ Setup Lighthouse CI

[Lighthouse](https://developers.google.com/web/tools/lighthouse) is very famous tool from Google for checking apps' performance, accessibility and a lot of others.

There is a brand new tool called [Lighthouse CI](https://github.com/GoogleChrome/lighthouse-ci) that is built up to help us checking our apps constantly and via our CI systems.

Our motivation for using such tool was to be aware of any decreases of our apps' performance, SEO functionality ...

Here are a few steps you need to go through to set up a project for using LHCI (Lighthouse CI).

* [Picking right mode](#picking-right-mode)
* [Install LHCI package](#install-lhci-npm-package)
* [Create a config file](#create-a-config-file)
* [Set server base url](#set-server-base-url)
* [Run a wizard](#run-a-wizard)
* [Set the token](#set-the-token)
* [Update package.json](#update-package.json)
* [Update Gitlab CI config](#update-gitlab-ci-config)
* [Update .gitignore](#update-.gitignore)
* [Run LHCI in your project](#run-lhci-in-your-project)
* [See the result](#see-the-result)

## Picking right mode

There are more modes in which LHCI can run and you have to pick correct one for your use case:

* *Local against static files* - implemented in Ackee's *react* pipeline, it runs in every pipeline. Part of project's repo
* *Local against server* - implemented in Ackee's *react-ssr* pipeline, it runs in every pipeline. Part of project's repo
* *Remote* - implemented in Ackee's *lighthouse* pipeline, it runs periodically against deployed version of application. Own repo

## Install LHCI npm package
Install [@lhci/cli](https://www.npmjs.com/package/@lhci/cli) globally into your computer to be able to run a wizard later.

```bash
yarn global add @lhci/cli
```

or

```bash
npm install -g @lhci/cli
```

## Create a config file
Create a file with name **.lighthouserc.js** in your root directory. And fill it with right code for your usecase:

### Local against static files mode

```js
module.exports = {
    ci: {
        assert: {
            assertions: {
                "offscreen-images": "off",
            },
        },
        collect: {
            staticDistDir: "./build/",
            numberOfRuns: 2,
            settings: {"chromeFlags": '--no-sandbox'}
        },
        upload: {
            target: "lhci",
            serverBaseUrl: "",
            token: ""
        },
    },
};
```

### Local against server mode

```js
const { REACT_APP_ENV } = process.env;

module.exports = {
    ci: {
        assert: {
            assertions: {
                "offscreen-images": "off",
            },
        },
        collect: {
            url: process.env.LHCI_URL,
            startServerCommand: `NODE_ENV=production REACT_APP_BUILD_ENV=${REACT_APP_ENV} node server/lib/index.js`,
            // https://github.com/GoogleChrome/lighthouse-ci/blob/master/docs/configuration.md#startserverreadypattern
            startServerReadyPattern: /Listening on/,
            numberOfRuns: 3,
            settings: {"chromeFlags": '--no-sandbox'}
        },
        upload: {
            target: "lhci",
            serverBaseUrl: "",
            token: ""
        },
    },
};
```

### Remote mode

```js
module.exports = {
    ci: {
        assert: {
            assertions: {
                "offscreen-images": "off",
            },
        },
        collect: {
            url: process.env.LHCI_URL,
            numberOfRuns: 2,
            settings: {"chromeFlags": '--no-sandbox'}
        },
        upload: {
            target: "lhci",
            serverBaseUrl: "",
            token: ""
        },
    },
};
```

*Note:* config was previously named **.lighthouserc.json**, it was renamed to **.lighthouserc.js** - .js is required, if file is not found, CI step will be automatically skipped

## Set server base url
Set *serverBaseUrl* field with value you can find in internal password manager as **lighthouse server url**.

## Run a wizard
Run wizard at you project root directory with this command:

```bash
lhci wizard
```

Then set the name of your project. Please use format *$repo-$mode*, so if you create LHCI project for pipeline (thus local) tests for `milacci-web` project, it will be:
`milacci-web-local`. If you want to create repo with periodic tests for this project, you will create repo named `milacci-web-lighthouse` and LHCI project will be named `milacci-web-lighthouse-remote`.
There is need to distinguish between *Local against static files mode* and *Local against server mode*.

Please be specific and do not make a mistake here!!! Lhci server has no chance to edit it when its once created for now.

After succesfull path through the wizard you will get specific **token** which you **MUST** copy from bash and use in next step.

## Set the token
Copy the **build token** from your wizard and paste it into the **.lighthouserc.js** config file in field *token*.

> **DON'T FORGET TO SAVE IT AND PASTE IT THERE. TOKEN IS NO LONGER REACHABLE THEN!**

## Save admin token

Go to internal password manager, create new record in form : lhci - $project_name (eg. *lhci - milacci-web-local*), Insert *admin token* as username and admin token value as password. Share with *frontend* and *devops* groups.

## Update package.json
### Add package
Add lhci/ci library into your **dependency** section in your *package.json* or simply install it as your local dependency with:

```bash
yarn add @lhci/cli -D
```

or

```bash
npm install @lhci/cli --save-dev
```

### Add run script
In your script section in *package.json* file add this script for running lhci:

```json
"ci-lighthouse": "lhci autorun"
```

> Our CI system uses this command during deploy but you can use this command when want to run it locally.

## Update Gitlab CI config

### All modes

Add variable with LHCI client's version to *.gitlab-ci.yaml* config file in project's root directory, eg.

```bash
variables:
  LHCI_VERSION: 0.6.0
```

*Note:* **IMAGE VERSION MUST BE EXACTLY SAME AS VERSION OF PACKAGE IN package.json, OTHERWISE PIPELINE CAN FAIL**


### Remote and Local against server modes

You must specify URL against which LHCI will run. You can do this on per branch basis in `ci-branch-config` folder.

So, for example if you have basic auth on development branch (and all you feature branches are created from development) and no basic auth on stage:

common.env:
```bash
LHCI_URL=http://basic_user:basicpassword@localhost:3000
```
*BEWARE*: Basic auth user and password CAN NOT contain special character, otherwise escaping can break them. Please use only alphabet symbols.

stage.env:
```
LHCI_URL=http://localhost:3000
```

## Update .gitignore
Add lighthouse built folder in your *.gitignore* file
```json
# Lighthouse
*.lighthouseci
```

## Run LHCI in your project
Run lhci collecting with this command:

```bash
yarn ci-lighthouse
```

or

```bash
npm run ci-lighthouse
```

## See the result
Go to our [lhci server](https://lhci.ack.ee/) and sign in with a credentials you can find in our password manager.

Choose your project there, check your results and share it with your team!
