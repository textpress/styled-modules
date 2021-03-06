# Styled Modules

[![Greenkeeper badge](https://badges.greenkeeper.io/traveloka/styled-modules.svg)](https://greenkeeper.io/)
[![Build Status](https://travis-ci.org/traveloka/styled-modules.svg?branch=master)](https://travis-ci.org/traveloka/styled-modules)
[![Coverage Status](https://coveralls.io/repos/github/traveloka/styled-modules/badge.svg?branch=master)](https://coveralls.io/github/traveloka/styled-modules?branch=master)
[![NPM Version](https://img.shields.io/npm/v/styled-modules.svg?style=flat-square)](https://www.npmjs.com/package/styled-modules)

CSS Modules support in **Next.js** based on [styled-jsx](https://github.com/zeit/styled-jsx).

## Installation

Styled Modules uses **CSS Loader** to enable **CSS Modules**, run the following to install it:

```bash
npm install css-loader --save-dev
```

Next, run the following to install Styled Modules:

```bash
npm install styled-modules
```

## Configuration

In the root of your **Next.js** project directory, create `next.config.js` with the following:

```js
// next.config.js
module.exports = (config, { dev }) => {
  config.module.rules.push(
    {
      test: /\.css$/,
      use: [
        'babel-loader',
        'styled-modules/loader',
        'css-loader?modules',
      ],
    },
  );
  return config;
};
```

Next, add `styled-modules/babel` to plugins in your `.babelrc`:

```json
{
  "plugins": [
    ["styled-modules/babel", {
      "pattern": "\\.css$"
    }]
  ]
}
```

## Usage

Now, you can start using **CSS Modules** in your **Next.js** project as follows:

```js
import styles from './styles.css';

export default () => (
  <div>
    <h1 className={styles.title}>Styled Modules</h1>
    <ol>
      {[
        'Item 1',
        'Item 2',
        'Item 3'
      ].map((item, index) => (
        <li key={index}>{item}</li>
      ))}
    </ol>
    {!this.props.hidden && (
      <div>Hidden content</div>
    )}
  </div>
);
```

Notice that the styles only got applied on the client side. To make it works on the server side as well, create a custom document at `./pages/_document.js` with the following code:

```js
import Document from 'next/document';
import flush from 'styled-modules/server';

class MyDocument extends Document {
  static getInitialProps({ renderPage }) {
    return {
      ...renderPage(),
      styles: flush(),
    };
  }
}

export default MyDocument;
```

## How It Works

- Check whether `styled-modules/style` is imported explicitly. If so, then it won't transpile. This will allow you to manually wrap your component with `styled-modules/style`. Note that you should only wrap the root of your component.
- Find any `import` and `require` statements that matches `pattern` option. If none is matched, then it won't transpile.
- Find any jsx syntaxes. if its parent is not a jsx syntax, then wrap it with `styled-modules/style`.

The example above transpiles to the following:

```js
import _StyledModules from 'styled-module/style';
import styles from './styles.css';

export default () => (
  <_StyledModules
    styles={[{
      __hash: styles.__hash,
      __css: styles.__css
    }]}
  >
    <div>
      <h1 className={styles.title}>Styled Modules</h1>
      <ol>
        {[
          'Item 1',
          'Item 2',
          'Item 3'
        ].map((item, index) => (
          <_StyledModules
            key={index}
            styles={[{
              __hash: styles.__hash,
              __css: styles.__css
            }]}
          >
            <li>{item}</li>
          </_StyledModules>
        ))}
      </ol>
      {!this.props.hidden && (
        <_StyledModules
          styles={[{
            __hash: styles.__hash,
            __css: styles.__css
          }]}
        >
          <div>Hidden content</div>
        </_StyledModules>
      )}
    </div>
  </_StyledModules>
);
```

As you can see, the `styled-modules/style` component is injected multiple times. This could be a problem when a wrapped component parent depends on or needs to manipulate the children, i.e. `react-transition-group`. To avoid that, you can explicitly import and wrap the component like the following:

```js
import _StyledModules from 'styled-modules/style';
import styles from './styles.css';

export default () => (
  <_StyledModules styles={[styles]}>
    <div>
      <h1 className={styles.title}>Styled Modules</h1>
      <ol>
        {[
          'Item 1',
          'Item 2',
          'Item 3'
        ].map((item, index) => (
          <li key={index}>{item}</li>
        ))}
      </ol>
      {!this.props.hidden && (
        <div>Hidden content</div>
      )}
    </div>
  </_StyledModules>
);
```

## License

MIT
