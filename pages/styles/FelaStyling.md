# Styling components

There is severeal ways of styling components using Fela

- [`FelaComponent`](#felacomponent)
- [`connect` HOC](#connect-hoc)
- [`useFela` hook](#usefela-hook)
- [`createComponent` and `createComponentWithProxy`](#createcomponent-and-createcomponentwithproxy)
- [jsx pragma `fe`](#jsx-pragma-fe)

Will's go through all of them and look at how use them and what are their advantages and disadvantages.

For all examples we assume file containing styles

```js
// styles.js
const buttonStyle = ({ big, color }) => ({
  backgroundColor: color,
  fontSize: big ? 18 : 15
});

const iconStyle = () => ({
  marginLeft: "0.5em"
});
```

But first let's show several general tricks for styling

## Class names

Fela generates in development poorly debuggable class names like `_137u7ef`. But there is a way how to change it and make class names meaningful like `Button_button_137u7ef`.

We need to just use [`fela-monolithic`](https://github.com/robinweser/fela/tree/master/packages/fela-monolithic) enhancer with `prettySelectors` option set to `true` when configuring Fela renderer.

```js
import { createRenderer } from 'fela';
import felaMonolithic from 'fela-monolithic';

const renderer = createRenderer({
  enhancers: [
    felaMonolithic({
        prettySelectors: process.env.NODE_ENV === 'development',
    }),
  ],
  ...
});
```

Unfortunatelly, that **works only for [`connect` HOC](#connect-hoc) and [`createComponent`](#createcomponent)**, other 3 methods will alway have meaningless class names.

## Definition

Usually we'll write styles as a functions, because it brings an advantage of passing props for modificating styles and using a theme.

```js
const buttonStyle = ({ big, theme }) => ({
  backgroundColor: theme.colors.red,
  fontSize: big ? 18 : 15
});
```

however if we don't need any of these features, we can define a style as a simple object

```js
const buttonStyle = {
  backgroundColor: "red",
  fontSize: 16
};
```

it's useful for prototyping and refactoring purposes rather than for daily use, because we value the consistency of writing styles and **functions are a preferred way**.

## Composition

Fela provides `combineRules` helper for composing more styles together

```js
import { combineRules } from "fela";

const formButtonStyle = combineRules(buttonStyle, formItemStyle);
```

however there is a syntax sugar definition

```js
const formButtonStyle = [buttonStyle, formItemStyle];
```

the two definitinos are equivalent.

---

## `FelaComponent`

```jsx
import { FelaComponent } from 'react-fela'
import * as styles from './styles';

const Button = ({ color, big = false, text, icon = null }) => (
  <FelaComponent style={styles.button} {...{ big, color }} as="button">
    {text}
    {icon && <FelaComponent style={styles.icon}>{icon}</FelaComponent>}
  </FelaComponent>
)

// usage
<Button color="red" text="Click me" />
```

- Each **`FelaComponent` renders `RenderProvider` and `ThemeProvider`** and as we use it more and more, components tree complexity grows.

- `FelaComponent` is rendered as a `div` element in default. We can change it with `as` prop however this property can be only string. If we want to render it as an another component we need to use "children as a render function" pattern

  ```jsx
  import { FelaComponent } from 'react-fela'
  import MyButton from '../MyButton'
  import Icon from '../icon'
  import * as styles from './styles';

  const Button = ({ color, big = false, text, iconType = "star" }) => (
    <FelaComponent style={styles.button} {...{ big, color }}>
      {({ className }) => (
        <MyButton className={className}>{text}</MyButton>

        <FelaComponent style={styles.icon}>
          {({ className }) => (
            <Icon className={className} type={iconType} />
          )}
        </FelaComponent>
      )}
    </FelaComponent>
  )
  ```

  As you may notice, nesting components styled this way tends to a messy code as components grow

- We need to pass modificator props like `big`, `color` etc. into every `FelaComponent` which style want to use them

### Overriding styles

To override component styles from other component, we need to implement a special property called for example `customStyle` and use it to combine styles together

```jsx
import { FelaComponent } from 'react-fela'
import * as styles from './styles';

const Button = ({ color, big = false, text, icon = null, customStyle }) => (
  <FelaComponent style={[styles.button, customStyle]} {...{ big, color }} as="button">
    {text}
    {icon && <FelaComponent style={styles.icon}>{icon}</FelaComponent>}
  </FelaComponent>
)

// Usage
const buttonInAppStyle = () => ({
  margin: 5,
})

<Button customStyle={buttonInAppStyle} big>Click me</Button>
```

- If we would like to override also icon style we had to provide it as another prop, eg. `iconCustomStyle` or maybe better, make `customStyle` an object like

  ```jsx
  const style = {
    button: buttonStyle,
    icon: iconStyle
  }

  <Button customStyle={style}>Click me</Button>
  ```

---

## `connect` HOC

```jsx
import { connect } from 'react-fela'
import * as buttonStyles from './styles';

const Button = ({ color, big = false, text, icon = null, styles }) => (
  <button className={styles.button} big>
    {text}
    {icon && <span className={styles.icon}>{icon}</span>}
  </button>
)

const ConnectedButton = connect(buttonStyles)(Button)

// Usage
<ConnectedButton color="red">Click me</ConnectedButton>
```

- We can easily compose the component using HTML elements or custom components without complex patterns
- HOC pass to the component also raw styles (before transformation to the css class) and fela theme

  ```jsx
  import { connect } from 'react-fela'
  import * as buttonStyles from './styles';

  const SuccessButton = ({ color, text, styles, rules, theme }) => {
    const extend = {
      icon: rules.button
    }

    return (
      <button className={styles.button} big>
        {text}
        <Icon type="check" extend={extend} />
      </button>
    )
  }

  const ConnectedButton = connect(buttonStyles)(Button)

  // Usage
  <ConnectedButton color="red">Click me</ConnectedButton>
  ```

### Overriding styles

To override component styles we dont need to define any special prop, because Fela `connect` do it for as. It's called **`extend`** and it's alway an object of styles keyed by same styles names as provided to the `connect`.

```js
import { connect } from 'react-fela'
import * as buttonStyles from './styles';

const Button = ({ color, big = false, text, icon = null, styles }) => (
  <button className={styles.button} big as="button">
    {text}
    {icon && <span className={styles.icon}>{icon}</span>}
  </button>
)

const ConnectedButton = connect(buttonStyles)(Button)

// Usage
const overridenIconStyle = () => ({
  border: '1px solid black',
})

const buttonExtend = {
  icon: overridenIconStyle,
}

<ConnectedButton extend={buttonExtend}>Click me</ConnectedButton>
```

- notice that we can omit the styles that are not needed to override (`button` style in our example)
- be sure to **pass an object of styles not just the style to `extend`**, otherwise you get a `Cannot assign to read only property '0' of string` error
- all props passed to Fela connected component are instantly provided to all styles, even to the extending styles

  ```jsx
  const buttonExtendStyle = ({ big }) => ({
    width: big ? 30 : 20,
  })

  const extend = {
    button: buttonExtendStyle
  }

  <ConnectedButton extend={extend} big />
  ```

---

## `useFela` hook

Hook returns css function that can transform styles to css class names

```js
import { useFela } from 'react-fela'
import * as styles from './styles';

const Button = ({ color, big = false, text, icon = null }) => {
  const { css } = useFela({ big, color })

  return (
    <button className={css(styles.button)} big as="button">
      {text}
      {icon && <span className={css(styles.icon)}>{icon}</span>}
    </button>
  )
}

// Usage
<Button color="red">
```

- object returned by `useFela` contains also `theme` for case we would need it
  ```js
  const { css, theme } = useFela({ big, color });
  ```

### Overriding styles

Same as for [`FelaComponent`](#felacomponent) - special prop must be supplied.

```jsx
import { useFela } from 'react-fela'
import * as styles from './styles';

const Button = ({ color, big = false, text, icon = null, customStyle }) => {
  const { css } = useFela({ big, color })
  return (
    <button className={css(styles.button, customStyle)}>
      {text}
      {icon && <span className={css(styles.icon)}>{icon}</span>}
    </button>
  )
}

// Usage
const buttonInAppStyle = () => ({
  margin: 5,
})

<Button customStyle={buttonInAppStyle} big>Click me</Button>
```

---

## `createComponent` and `createComponentWithProxy`

Using `createComponent` we can quickly create a component without any JSX needed.

```js
import { createComponent } from "react-fela";
import * as styles from "./styles";

const Button = createComponent(styles.button, "button", ["onClick"]);

// Usage
<Button big onClick={handleClick}>
  Click me
</Button>;
```

You may notice that we have to define which props we want to pass through to the `button` as a third parameter.  
Instead of static array of props names, we can use a function that receives component props and returns the array of props names.

```js
// pass only `onClick` prop to the button
const Button = createComponent(styles.button, "button", ["onClick"]);

// pass all props that are set on Button to the button, this set also style modificators
const Button = createComponent(styles.button, "button", Object.keys);

// same as in first example, pass only `onClick` prop
const Button = createComponent(styles.button, "button", props => ["onClick"]);
```

However there is even smarter way of proxying props. It's a `createComponentWithProxy` function and is useful if you don't know all the props you want to pass. It pass only those props which are not used in styles. If we need some prop to be used styles but still passed to the underlying component we can use the 3rd argument

```js
const Button = createComponentWithProxy(styles.button, 'button');
const Button2 = createComponentWithProxy(styles.button, 'button', ['disabled']);

// Pass `type` and `onClick` to the button while `color` and  `big` are ommited
<Button type="submit" onClick={handleClick} color="red" big>You cant click me</Button>

// Pass `type` and `disabled` to the button while `color` is ommited
<Button2 type="submit" color="red" disabled>Click me</Button>
```

- `createComponent` creates [`FelaComponent`](#felacomponent) under the hood
- We can't create a complex component structure using `createComponent` therefore it's more convinient for simple atomic components

### Overriding styles

Component created with `createComponent` or `createComponentWithProxy` can receive `extend` prop as well as component created with [`connect` HOC](#connect-hoc).

```js
import { createComponent } from 'react-fela'
import * as styles from './styles';

const Button = createComponent(styles.button, 'button');

// Usage
const formButtonStyle = () => ({
  color: 'white',
})

<Button extend={formButtonStyle}>Click me</Button>
```

---

## jsx pragma `fe`

Pragma is a compiler directive, JSX pragma tells [how to transpile JSX elements](#https://www.gatsbyjs.org/blog/2019-08-02-what-is-jsx-pragma/). In default Babel transpile all JSX elements to call of `React.createElement` but this behaviour can be changed using JSX pragma.

```jsx
import React from "react";

<div />;

// is transpiled into

React.createElement("div");
```

we can change it with special directive written in comment

```jsx
/* @jsx fe */
import { fe } from "react-fela";

<div />;

// is transpiled into

fe("div");
```

This allows us to use a special prop `css` on any JSX element. What happens under the hood is that `fe` create a
[`FelaComponent`](#felacomponent) and pass a style into it.

```js
/* @jsx fe */
import { fe } from 'react-fela'
import * as styles from './styles';

const Button = ({ color, text }) => (
  <button css={styles.button}>
    {text}
    {icon && <span css={styles.icon}>{icon}</span>}
  </button>
)

// Usage
<Button>Click me</Button>
```

- **we can't use style modificators** like `color` or `big` in styles because none prop is supplied into them. That's because Fela has no way how to
  distinguish between regular and style modification prop and therefore doesn't know which prop should be avoided to set on the element.
