# Custom Gatsby SSR

When we use normal gatsby SSR render we can get into trouble 
with React Intl and it's rich formatting. It ignores our html tags into messages so when 
user without JS visits our page then he will html tags, which is not we want.

Solution is to use DOM parser which is not available during build time and is available
only in browser. During the build process in `build-html` (production) and `develop-html` the script
will look up if there is any DOM parser and if it is not, he will not parse it anymore. That is the reason why we
see these tags.

## Solution

1) First install JsDOM dependency
`npm install jsdom`

2) Edit `gatsby-ssr.js` which is located in the root project directory
    - append `global.DOMParser = new (require('jsdom').JSDOM)().window.DOMParser;` at the beginning of the file
    - now the compiler can use our DOM parser
    - but there is a pitfall, during the `build` process we get some sort of error messages about `missing canvas`, we will solve
    these problem in next step. 

3) Edit `gatsby-node.js` and fix these problems by ignoring request to canvas, which our library cannot handle.
   So for every request to canvas we just return an empty module. 

    ```
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

4) Now our rich text formatting will be parsed during the build time.