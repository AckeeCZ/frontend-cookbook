# 🎨 Using Fela - tips & tricks

Using [Fela](http://fela.js.org) for components styling is easy and pretty straightforward. But time to time we find that something that was easy to make in CSS with preprocessors isn't such obvious now. Here we should collect short useful recipes to prevent stuck in development because of some problem in basic things.

- [Manipulation colors](#manipulating-colors)
- [Animations using keyframes](#animations-using-keyframes)

## Manipulating colors

As we were using CSS preprocessory we used to use standard functions for manipulating colors like `darken`, `lighten` `rgba` (for setting alpha value) and so on. But Fela (and CSS in JS solutions generally) doesn't have such built-in functions, so how we could manipulate colors when using it? There is a tiny library called [`tinycolor2`](https://www.npmjs.com/package/tinycolor2) that is perfect for that purpose

There is a few examples from the wild

```js
import tinyColor from "tinycolor2";

export const buttonStyle = ({ theme }) => ({
  color: theme.colors.grey,
  backgroundColor: tinyColor(theme.colors.grey)
    .darken(30)
    .toString()
});
```

```js
import tinyColor from "tinycolor2";

export const menuItemStyle = ({ theme }) => ({
  backgroundColor: theme.colors.black,

  ":hover": {
    backgroundColor: [
      theme.colors.grey, // this is a fallback value for the browsers which doesn't colors with alpha value
      tinyColor(theme.colors.black)
        .setAlpha(0.5)
        .toRgbString()
    ]
  }
});
```

But there is more, just have a look into [the documentation](https://www.npmjs.com/package/tinycolor2#methods) and then enjoy using it.

## Animations using [keyframes](https://developer.mozilla.org/en-US/docs/Web/CSS/@keyframes)

Our frontend developer Jirka has done a great job researching how keyframe animations work in Fela and has made a React component for using them with any other component.

```jsx
import React from "react";
import PropTypes from "prop-types";
import { RendererContext, ThemeContext } from "react-fela";

function FelaKeyframe({ children, keyframe, ...props }) {
  const renderer = React.useContext(RendererContext);
  const theme = React.useContext(ThemeContext);
  const animationName = React.useMemo(
    () => renderer.renderKeyframe(keyframe, { ...props, theme }),
    [keyframe, renderer, theme]
  );
  return children(animationName);
}

FelaKeyframe.propTypes = {
  children: PropTypes.func.isRequired,
  keyframe: PropTypes.func.isRequired
};

export default FelaKeyframe;
```

The component uses a [render props pattern](https://www.robinwieruch.de/react-render-props-pattern/) and it's easy to use.

Let's say we want to create a loader component. It should be spinning shape, first we create the styles

```js
export const spinner = ({ theme: { colors }, animationName }) => ({
  // definition of loader shape, it can be everything you want
  width: "60px",
  height: "60px",
  margin: "1rem auto",
  borderRadius: "50%",
  backgroundColor: "transparent",
  border: `2px solid ${colors.lightGrey}`,
  borderTopColor: colors.text,
  borderBottomColor: colors.text,

  // this makes the animation, animationName is supplied from props and made by FelaKeyFrame
  animation: `1000ms ${animationName} 'linear' infinite`
});

// keyframe definition that will be passed to FelaKeyFrame
export const spin = () => ({
  "0%": {
    transform: "rotate(0deg)"
  },
  "100%": {
    transform: "rotate(360deg)"
  }
});
```

then we use the `FelaKeyFrame` component to animate the loader

```jsx
import React from "react";
import PropTypes from "prop-types";
import { FelaComponent } from "react-fela";
import FelaKeyFrame from "./FelaKeyFrame";

import * as styles from "./Loader.styles";

const Loader = ({ text }) => (
  <FelaKeyframe keyframe={styles.spin}>
    {animationName => (
      <FelaComponent style={styles.spinner} animationName={animationName}>
        {({ className }) => <div className={className} title={text} />}
      </FelaComponent>
    )}
  </FelaKeyframe>
);

Loader.propTypes = {
  text: PropTypes.string
};

Loader.defaultProps = {
  text: ""
};

export default Loader;
```

and now the loader is spinning, awesome!
