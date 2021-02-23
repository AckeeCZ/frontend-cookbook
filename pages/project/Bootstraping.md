# Bootstraping project

Because we are usual lazy programmers, we like to make things as automated and done out of the box as possible.

You can choose from guides of two project types:

- [Dynamic React application](#react-application)
- [Gatsby generated static website](#gatsby-website)

## React Application

React applications are bootstraped with our custom fork of create-react-app by Facebook - see [github.com/ackeeCZ/create-react-app](https://github.com/ackeeCZ/create-react-app).

**To create a new project call following command:**

```bash
 npx @ackee/create-react-app  my-app
```

### Project structure

Created [source files structure](https://github.com/AckeeCZ/create-react-app/tree/master/packages/react-scripts/template/src) is easy to understand, but it's worth to describe the files structure and some patterns we're trying to follow

#### `components`, `containers`, `HOC`

These folders are rarely used and contains globals components, used across all app, hardly assignable to some module.

#### `services`

Contains sources that often relates to `redux` like selectors, reducers, sagas, etc. Not intended to contain static sources, like config, constants, etc. but rather functional stuff.

#### `translations`

Contains text translations of the app.

#### `styles`

Intended for global app styles, contains

- app theme definition (colors, fonts, sizes, etc.).
- vendor styles (`antd`, 3rd party components styles).
- custom styles for overriding vendor styles globally (eg. restyle `antd` input).

#### `modules`

There is contained most of the application code, it's separated into folders which represents logical areas of code and can cooperate.

There are 2 or 3 modules in default - `application`, `auth` and `core` (`auth` module is optional and included if your request it at bootstraping phase). Structure of each module is similar to application root with two important differences:

- Module exposes all its sources, needed by other modules or rest of application through `index.js` file.
  You should every time import from module root and **never break contracts** with imports leading deeper into the module.

  ```js
  // Good
  import { MyComponent } from "./modules/my-module";
  // Bad
  import MyComponent from "./modules/my-module/components/MyComponent";
  ```

- Each dependency from outside of the module is imported in module's `dependencies.js` file and every file inside the module should import them from there. **No other file than `dependencies.js` should contain import that leads outside of the module.**
  ```js
  // Good
  import { React } from "../dependecies";
  import Button from "../components/Button";
  // Bad
  import React from "react";
  ```

Example structure of module looks like

```
modules/
  |- application/
  |- core/
  |- auth/
   - my-module/
     |- components/
     |- config/
     |- constants/
     |- containers/
     |- services/
       |- sagas/
       |- actionTypes.js
       |- actions.js
       |- reducer.js
        - selectors.js
     |- dependencies.js
      - index.js
```

### New modules

To make creating modules easier, the project has npm script command (that uses code generator [Hygen](https://www.hygen.io) under the hood) you can use, just run

```bash
npm run create-module app-layout
```

where `app-layout` is name of the module you wanna create. Then look into `modules` directory when new folder `my-module` should appear.

## Gatsby website

Better than start from the scratch is to base new project on one of provided starters. Assume you have `gatsby-cli` already installed (if not, run `npm install -g gatsby-cli`) then execute

```bash
gatsby new my-website https://github.com/gatsbyjs/gatsby-starter-hello-world
```

where `my-website` is name of the website you're going to create and `https://github.com/gatsbyjs/gatsby-starter-hello-world` is repository url of starter.

If you need multi language website you would better [base it on our starter](https://medium.com/@marek.janca/840c27795827) for internatialized websites

```bash
gatsby new my-multi-lang-website https://github.com/AckeeCZ/gatsby-starter-internationalized
```
