# Code style onboarding
This guide should cover most of the stuff that is taken into consideration during code reviews.

Should anything be unclear, [contact me here](https://strv.slack.com/messages/@matrinpetyrek/).


## Editor setup
recommended is https://code.visualstudio.com/
required extensions:
  - editor config
  - eslint
  - prettier
  - vscode-styled-components

## Initializing project
- https://github.com/facebook/create-react-app

- make it your own - clean unused files, customize readme etc.

- setup eslint - https://github.com/strvcom/eslint-config-javascript

- use git, work in `dev` branch and make it the default branch

## Project structure
This is probably the most important part, because it dictates how maintainable the project will be.
Unless you have a strong opinion about doing something otherwise, you should't make a mistake be following the following structure

```
/
  .env
  .env-example
  src/
    /pages
      /urls.js - contains routes, that way you won't have magic string constants that can break when you change route naming
      /ContactPage
        /index.js - no styled components here
        /styled.js - named exports of all styled components
        /ContactPageSpecificComponent
          /index.js - a component that is only used in /ContactPage
          /styled.js
          /AnotherComponent
            /index.js - this can be arbitrarily nested
      /AboutUsPage
        ...
    /components - only "global" dumb components that are used in multiple places
      /Button
        /index.js
        /styled.js
      ...
    /containers - only "global" smart components that are used in multiple places
      /ConnectedComponent
        /index.js
        /styled.js
      ...
    /componentsStyled
      /Typography.js - exports typographic styled components that are reused everywhere, such as `Title, Paragraph` and what not.
    /data - contains logic files related to data, such as redux files etc. Should be split per entites
      /Animal
        actions.js
        selectors.js
        reducer.js
        epics.js - redux observable epics
        helpers.js - helper functions related to this entity
        types.js - Flow types for the entities here
        /graphql - if you have GQL
          /fragments.js
          /queries.js
          /mutations.js
    /common - helper functions that dont fit elsewhere
      ...
    /types - non-library typings related to the whole project
    /hocs - higher order components
    /global - global configuration files
      config.js
      global-styles.js
      theme.js
      redux/
        store.js
        reducers.js
        persistor?
        ...middlewares?
```
*No file should be over 100 lines of code*, if it is, you should split something into multiple components. I'd say `styled.js` is an exception.
If you have structure like this
```
/something
  /somethingElse.js - where this file is too long
```
Simply make `somethingElse` a directory and spread the contents across multiple files!


## Components
Prefer stateless components whenever possible.
```
const Foo = props => <div>hello world</div>
```
If you need some handlers/state prefer [recompose](https://github.com/acdlite/recompose).
```
import { compose, branch, renderComponent, withHandlers } from 'recompose'

const Bar = ({ handleClick }) => <div onClick={handleClick}>hello world</div>

const enhancer = compose(
  branch(props => props.loading, renderComponent(Loader)),
  withHandlers({
    handleClick: props => e => {
      console.log("clicked")
      props.something(e.target.value)
    }
  })
)

export default enhancer(Bar)
```

## Imports
Don't use wildcard imports
`import * as Foo from 'bar'`. It is unfriendly to tree shaking and can bloat your bundle size.

## Packages
This is a two-edged weapon. You don't want to deal with complex logic if well tested package exists. You also don't want to deal with low level code, that is difficult to get right across different browsers, so using a package would be advised.
Do remember though, that all the libraries increase the bundle size and will make loading of the app longed, which can ultimately lead to poor user experience.

## Recommended libraries
You pretty much always want these
- `recompose` - compilation of usefull HoCs
- `ramda` - functional utility library
- `redux-observable` - for handling side effects in your redux based application. If you are used to sagas, this will do you the same service, but will teach you RxJS, which is usefull, unlike generators. [Learn about them here](https://github.com/petyrek/guides/blob/master/redux-observable.md). You may also need to know [RxJS](https://github.com/petyrek/guides/blob/master/rxjs.md).
- `formik` and `yup`- to handle forms

*Discouraged libraries*
- immutableJS - you probably don't need it
- redux-saga - generators deserve to die, prefer `redux-observable`
- redux-form - use `formik` instead
- lodash - use `Ramda` instead

## Assets
Use appropriate formats to reduce size.
For non-transparent images always `jpg`. When transparency is required `png` is your friend. 

For icons use svg and then:
  - minimize using https://jakearchibald.github.io/svgomg/
  - create a React component out of them

You can then take advantage of `props` to alter their properties
```js
import React from 'react'
export default props =>
  <svg>...</svg>
```

For fonts
  - use https://github.com/bramstein/fontfaceobserver 
```js
var font = new FontFaceObserver('My Family', { weight: 400 });

font.load().then(
  () => console.log('Font is available'),
  () => console.log('Font is not available')
);
```
  - prefer google fonts if possible
  - for importing your own see https://stackoverflow.com/questions/41676054/how-to-add-fonts-to-create-react-app-based-projects


If you are having same font with different types (let's say Roboto-Regular, Roboto-Bold, Roboto-Italic) always use the same font-family in CSS, but change font with font-weight/font-style properties, like so:
```css
@font-face {
    font-family: 'Roboto';
    src: url('Roboto-regular-webfont.woff2') format('woff2'),
         url('Roboto-regular-webfont.woff') format('woff');
    font-weight: normal;
    font-style: normal;
}

@font-face {
    font-family: 'Roboto';
    src: url('Roboto-bold-webfont.woff2') format('woff2'),
         url('Roboto-bold-webfont.woff') format('woff');
    font-weight: bold;
    font-style: normal;
}

@font-face {
    font-family: 'Roboto';
    src: url('Roboto-regular-italic-webfont.woff2') format('woff2'),
         url('Roboto-regular-italic-webfont.woff') format('woff');
    font-weight: normal;
    font-style: italic;
}
```

## Static type checking
Forget about prop-types, it is a waste of time. Use [Flow](https://flow.org/en/) instead. Replacing prop-types with it is trivial, more complex uses are [explained here](https://github.com/petyrek/guides/blob/master/flow.md).


