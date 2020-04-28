# ⛑ Setup Lighthouse CI

[Lighthouse](https://developers.google.com/web/tools/lighthouse) is very famous tool from Google for checking apps' performance, accessibility and a lot of others.

There is a brand new tool called [Lighthouse CI](https://github.com/GoogleChrome/lighthouse-ci) that is built up to help us checking our apps constantly and via our CI systems.

Our motivation for using such tool was to be aware of any decreases of our apps' performance, SEO functionality ...

Here are a few steps you need to go through to set up a project for using LHCI (Lighthouse CI).

* [Install LHCI package](#install-lhci-npm-package)
* [Create a config file](#create-a-config-file)
* [Set server base url](#set-server-base-url)
* [Run a wizard](#run-a-wizard)
* [Set the token](#set-the-token)
* [Update package.json](#update-package.json)
* [Update Jenkinsfile](#update-jenkinsfile)
* [Update .gitignore](#update-.gitignore)
* [Run LHCI in your project](#run-lhci-in-your-project)
* [See the result](#see-the-result)

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
Create a file with name **.lighthouserc.json** in your root directory. And fill it with this code:

```json
{
    "ci": {
        "assert": {
            "assertions": {
                "offscreen-images": "off"
            }
        },
        "collect": {
            "numberOfRuns": 2
        },
        "upload": {
            "target": "lhci",
            "serverBaseUrl": "",
            "token": ""
        }
    }
}
```

## Set server base url
Set *serverBaseUrl* field with value you can find in internal password manager as **lighthouse server url**.

## Run a wizard
Run wizard at you project root directory with this command:

```bash
lhci wizard
```

Then set the name your project. Please be specific and do not make a mistake here!!! Lhci server has no chance to edit it when its once created for now.

After succesfull path through the wizard you will get specific **token** which you **MUST** copy from bash and use in next step.

## Set the token
Copy the **token** from your wizard and paste it into the **.lighthouserc.json** config file in field *token*. 

> **DON'T FORGET TO SAVE IT AND PASTE IT THERE. TOKEN IS NO LONGER REACHABLE THEN!**

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
"ci-lighthouse": "./node_modules/@lhci/cli/src/cli.js autorun"
```

> Our CI system uses this command during deploy but you can use this command when want to run it locally.


## Update Jenkinsfile
Add this parameter into your *Jenkinsfile* file next to other *run* parameters in your project root directory.
```bash
runLighthouse = true
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
