# style9

CSS-in-JS compiler based on the ideas of
[Facebook's stylex](https://www.youtube.com/watch?v=9JZHodNR184)

## Features

* Compiles to atomic CSS
* Typed styles using TypeScript
* Converts font-size to REM[¹](https://betterwebtype.com/articles/2019/06/16/5-keys-to-accessible-web-typography/)
* No selectors

## Install

```sh
# Yarn
yarn add style9

# npm
npm install style9
```

## Usage

```javascript
import style9 from 'style9';

// Styles are created by calling style9.create
const styles = style9.create({
  blue: {
    color: 'blue',
  },
  red: {
    color: 'red'
  },
  boldYellow: {
    color: 'yellow',
    fontWeight: 'bold'
  }
});

// `styles` is now a function which can be called with the keys of the style
// object, and returns a string of class names.
// Ternary operator and logical AND is supported
document.body.className = styles('blue', isRed && 'red');

// Styles are merged like `Object.assign`, with later styles taking precedent
styles('blue', 'red') // will be red
styles('red', 'blue') // will be blue
styles('red', 'blue', 'boldYellow') // will be yellow and bold

// `styles` can also be called with an object of booleans. Later keys take precedent
styles({
  blue: isBlue,
  red: isRed
});

// To combine with external styles, style9 can be called with the style objects
// Styles are merged like above
style9(styles.blue, otherStyles.yellow);

// Styles have to be statically defined, but constants are supported
const RED = 'red';

const moreStyles = style9.create({
  red: {
    color: RED
  },
  margin: {
    // All properties are written in camelcase
    // Integers are converted to pixels where appropriate
    marginTop: 8
  },
  longhands: {
    // Because of how classnames are resolved,
    // shorthand CSS properties are not supported
    // background: 'red' // INVALID
    backgroundColor: 'red', // OK
    // Some shorthand properties can be automatically expanded
    borderColor: 'blue' // will make all sides blue
    // Supported shorthands include borderColor, borderRadius, borderStyle,
    // borderWidth, margin, overflow, overscrollBehavior, and padding
    // See Style.d.ts for a complete list of supported CSS properties
  },
  padding: {
    // Longhands take precedent over shorthands
    // Will resolve to '12px 12px 12px 18px'
    paddingLeft: 18,
    padding: 12
    // Shorthand values will be copied to longhands which means
    // `padding: '12px 18px'` etc. is not supported
  },
  text: {
    // Font size is converted to REMs to follow users settings
    fontSize: 14
  },
  fadeIn: {
    // Animation names are created by calling style9.keyframes
    animationName: style9.keyframes({
      from: {
        opacity: 0
      },
      to: {
        opacity: 1
      }
    }),
    animationDuration: '1s'
  },
  mobile: {
    // Media queries are supported as well
    // They will be sorted mobile-first
    // NOTE: Media queries are not supported in TypeScript due to issue #17867
    '@media (min-width: 800px)': {
      display: 'none'
    }
  },
  pseudo: {
    // Pseudo-classes and elements can be used
    ':hover': {
      // They can be nested, as can media queries
      ':active': {
        '::before': {
          content: 'attr(title)'
        }
      }
    }
  }
});
```

## Options

- `minifyProperties` - minify the property names of style objects. Unless you
  pass custom objects to `style9()` not generated by `style9.create()`, this is
  safe to enable and will lead to smaller JavaScript output but mangled
  property names. Consider enabling it in production. Default: `false`

## Babel

```javascript
const babel = require('@babel/core');

const output = babel.transformFile('./file.js', {
  plugins: [['style9/babel', { /* options */ }]]
});
// Generated CSS
console.log(output.metadata.style9);
```

## Rollup

```javascript
import style9 from 'style9/rollup';

export default {
  // ...
  plugins: [
    style9({
      // include & exclude are supported according to rollup conventions
      // Either fileName or name is required. It will be passed to emitFile
      // fileName: unique name to for output file
      // name: name to use for output.assetFileNames pattern
      // parserOptions: options to pass to the Babel parser
      // ...Options
    })
  ]
};
```

## Webpack

```javascript
const Style9Plugin = require('style9/webpack');
const MiniCssExtractPlugin = require('mini-css-extract-plugin');

module.exports = {
  module: {
    rules: [
      {
        test: /\.(tsx|ts|js|mjs|jsx)$/,
        use: Style9Plugin.loader,
        options: {
          // parserOptions: options to pass to the Babel parser
          // ...Options
        }
      },
      {
        test: /\.css$/i,
        use: [MiniCssExtractPlugin.loader, 'css-loader']
      }
    ]
  },
  plugins: [
    new Style9Plugin(),
    new MiniCssExtractPlugin()
  ]
};
```

## Next.js

```javascript
const withTM = require('next-transpile-modules')(['style9']);
const withStyle9 = require('style9/next');

module.exports = withStyle9({
  // parserOptions: options to pass to the Babel parser
  // ...Options
})(withTM());
```

## Gatsby

```javascript
module.exports = {
  plugins: [
    {
      resolve: 'style9/gatsby',
      options: {
        // parserOptions: options to pass to the Babel parser
        // ...Options
      }
    }
  ]
}
```

## Vue.js

There is [an example repo](https://github.com/johanholmerin/style9-vue-example) you can see to get started.

## CSS Custom Properties with TypeScript

When using CSS properties not included in the [type definition](Style.d.ts), such as [CSS
Custom Properties](https://developer.mozilla.org/en-US/docs/Web/CSS/--*),
TypeScript will error. There are two ways of dealing with this:

1. Module augmentation. This will affect every use.
```typescript
declare module 'style9/Style' {
  interface StyleProperties {
    '--bg-color'?: string;
  }
}
```

2. Type assertion
```typescript
const styles = style9.create({
  lightTheme: {
    ["--bg-color" as any]: "#CCC"
  }
});
```
