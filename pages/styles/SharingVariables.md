# 🤝 Sharing variables between CSS & JS

Sometimes you have some value, let's say color definition or size of an element, that you need in styles and also Javascript sources. For that you would need to use some styles syntax that supports variables (eg. [SASS](https://sass-lang.com/) or [Less](http://lesscss.org/)). This recipe is about **sharing these variables in webpack based app**.

It doesn't matter if you need the values for inline styles or [Fela](http://fela.js.org) rule or theme. The main goal is to set style sources to export variables so they can be imported in Javascript. The main key to enable shared variables is [CSS Modules](https://github.com/css-modules/css-modules), especially their [interoperability specification](https://github.com/css-modules/icss#export).

## Setup webpack

Add [`css-loader`](https://github.com/webpack-contrib/css-loader) for webpack rules that load the style sources, and enable `exportOnlyLocals` option.

```js
module.exports = {
  module: {
    rules: [
      // loader for SASS styles
      {
        test: /\.scss$/,
        use: [
          'style-loader',  
          {
              loader: 'css-loader',
            options: {
                exportOnlyLocals: true
            }
          },
          'sass-loader'
        ]
      },
      // loader for Less styles
      {
        test: /\.less$/,
        use: [
          'style-loader',  
          {
            loader: 'css-loader',
            options: {
              exportOnlyLocals: true
            }
          },
          'less-loader'
        ]
      }
    ]
  }
};
```

`style-loader` plays no role for desired sharing capability and can be removed if you load styles other way, eg. using Fela.

## Use variables

Let's assume we have following styles sources:

```scss
// styles/header.scss
$header-height: 24;
$header-height-px: 24 + 0px;

.header {
    height: $header-height-px;
}

:export {
    height: $header-height;
}
```

What we've done when defining variables is to allow export of header height as a pure number value while preserving its value with unit for styles. It can be done also other way, for example using [function to strip the value](https://css-tricks.com/snippets/sass/strip-unit-function).

> Note: I haven't found a way how to export variables when using `.sass` syntax yet.

```less
// styles/content.less
@content-bg-color: #FF8CA2;

.content {
    background-color: @content-bg-color;
}

:export {
    bgColor: @content-bg-color;
}
```

And if you wanna use it in javascript source just do

```js
// index.js
import headerStyles from "./styles/header.scss";
import contentStyles from "./styles/content.less";

headerStyles.height === '24'; // true
contentStyles.bgColor === '#FF8CA2'; // true
```

## Utilize it with Ant design

This is an example of how to utilize sharing variables approach when you use 3rd party library with own styles and need to reuse its variables in your styles. 

Let's say we use [Antd](https://ant.design) (we really do) in our app, and we need to use some of its variables in styles for our component. Maybe we use Fela for styling components and and height of the [header](https://ant.design/components/layout/) is important to set logo height. 

Start with instructions described above to [setup webpack](#setup-webpack) and don't forget to set `javascriptEnabled` option to true for `less-loader`, its [needed for Antd styles](https://ant.design/docs/react/customize-theme#Customize-in-webpack), because it's less styles contain javascript. 

Then create file `variables.less` (since Antd uses Less syntax for styles) and fill it

```less
// styles/variables.less
@import '~antd/lib/style/themes/default.less';

:export {
    // passing the variable to unit causes change of "36px" to "36"
    layoutHeaderHeight: unit(@layout-header-height);
}
```

and then use it in the `Logo` component

```js
// components/Logo.jsx
import { FelaComponent } from 'react-fela';
import variables from '../styles/variables.less';

const logoRule = () => ({
    height: variables.layoutHeaderHeight,
});

const Logo = () => (
    <FelaComponent style={logoRule}>
        ...
    </FelaComponent>
);
```

That's the easy and comfortable way of leveraging variables sharing in your app.

## Resources

1. https://www.bluematador.com/blog/how-to-share-variables-between-js-and-sass
2. https://github.com/css-modules/icss#export
3. https://ant.design/docs/react/customize-theme