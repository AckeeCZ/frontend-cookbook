# Rich text formatting in Gatsby

---
**NOTE**

In version 4.0 React Intl does not lookup for `DOMParser` anymore and uses inner mechanism. So this guide applies prior to version `4.0`

---

# Description of the problem

When using Gatsby with React Intl and it's rich text formatting we will find out that 
our build HTML files contains escaped HTML tags which we have used in our rich text formatting (we can see them as strings). If Javascript is enabled (on client side) hydration will occur and these HTML tags will be treated as elements, not as strings anymore. But if Javascript is not enabled, user will see these elements as strings, which is not desirable.

Solution is to polyfill `DOMParser` object during server side rendering.

## Solution

During the build process in `build-html` (production) and `develop-html` the script
will look up if there is `DOMParser` and if it is not, he will not parse it anymore. That is the reason why we
see these tags.

1) First install JsDOM dependency
`npm install jsdom`

2) Edit `gatsby-ssr.js` (located in the root project directory)
* append `global.DOMParser = new (require('jsdom').JSDOM)().window.DOMParser;` at the   beginning of the file
* but there is a pitfall, during the `build` process we get some sort of error messages about `missing canvas`, we will solve that problem in the next step. 

3) Edit `gatsby-node.js` and fix `missing canvas` problem by ignoring requests to `canvas`, which our library cannot handle.

   So for every request to canvas we just return an empty module. 

```js
exports.onCreateWebpackConfig = ({ stage, rules, loaders, plugins, actions }) => {
    if (stage === 'build-html' || stage === 'develop-html') {
        actions.setWebpackConfig({
            module: {
                rules: [
                    {
                        test: /canvas/,
                        use: loaders.null(),
                    },
                ],
            },
        });
    }
};
```