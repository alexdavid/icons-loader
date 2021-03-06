# Icons loader (generate iconfonts for webpack)

This is a [webpack](https://github.com/webpack/webpack) loader for generating an iconfont from SVG dependencies.

Based on [iconfont-loader](https://www.npmjs.com/package/iconfont-loader) by [Jussi Kalliokoski](https://github.com/jussi-kalliokoski) thanks and <3.

It uses [gulp-iconfont](https://www.npmjs.com/package/gulp-iconfont) to create the font.

## Features

- Automatically generates fonts (iconfont) from `.svg` files
- Webpack loader + plugin for seamless workflow integration

## Installation

```
npm install icons-loader --save-dev
```

## Basic usage

Add the loader and plugin to your webpack config:

```javascript
import IconsPlugin from 'icons-loader/IconsPlugin'

const RUN_TIMESTAMP = Math.round(Date.now() / 1000)

const webpackConfig = {
  loaders: [{
      test: /\.svg$/,
      loader: 'icons-loader',
  }],
  plugins: [
    new IconsPlugin({
      fontName: 'icons',
      timestamp: RUN_TIMESTAMP,
      normalize: true,
      formats: ['ttf', 'eot', 'woff', 'svg']
    })
  ]
}
```

Now you can require the icons in your code:

```javascript
import iconFont from 'icons-loader'
import menu from './menu.svg'

console.log(iconFont) /*
{
  css: '@font-face(...)', // you could inject this into your body by using style-inject package?
  fontName: 'icons',
  glyphs: [1],
  <generated_icon_id>: {
    character: 'ea01',
    fontName: 'icons',
    unicode: ['']
  } ...
}
*/

console.log(menu) /*
{
  character: 'ea01',
  fontName: 'icons',
  unicode: ['']
}
*/
```

## Example workflow integration

So how can you integrate `icons-loader` into your webpack workflow? Here is how I use it:

- Create an `icons` directory in your project `src`
- Add `.svg` icons to the `icons` directory, check out these websites for free and excellent `.svg` icons:
  - [Noun Project](https://thenounproject.com/)) (recommended)
  - [UX Repo](http://uxrepo.com/)
- Create an `index.js` file in the new `icons` directory:

```javascript
import menu from './menu.svg'
import cross from './cross.svg'

export default {
  menu: menu,
  cross: cross
}
```

- Create an `icon` component:

src/components/icon.scss
```css
.icon {
  font-weight: normal;
  font-style: normal;
  font-decoration: none;
  text-transform: none;
  vertical-align: middle;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
}
```

src/components/icon.js
```javascript
import React, { Component, PropTypes } from 'react'

import styles from './icon.scss'
import * as icons from 'icons'

export default class Icon extends Component {
  static propTypes = {
    name: PropTypes.string.isRequired
  }

  render () {
    let icon = icons[this.props.name]

    if (icon === undefined) {
      console.warn('Unknown icon: ' + icon)
      return
    }

    return (
      <span
        className={styles.icon}
        style={
          fontFamily: icon.fontName
        }>
        {icon.unicode}
      </span>
    )
  }
}
```

- Use the icon component in other components like so:

src/components/header.js
```javascript
<Icon name='icon-file-name' />
<Icon name='menu' />
```

- Finally use [style-inject](https://www.npmjs.com/package/style-inject) or similar package to inject the css returned from `import iconFont from 'icons-loader'` in your body

The best way to do this is in your main `app` component. For example:

```javascript
import React from 'react'
import { render } from 'react-dom'

...

import styleInject from 'style-inject'
import iconFont from `icons-loader`

const injectIconFont = function () {
  styleInject(iconFont.css)
}

injectIconFont()

...

```

## Options

### Plugin

* `filenameTemplate` naming options for the font assets
* `filenameTemplate.name` the template to use. See [loader-utils docs](https://github.com/webpack/loader-utils#interpolatename)
* `filenameTemplate.regExp` the regexp passed to `loader-utils`

You can also add [gulp-iconfont](https://www.npmjs.com/package/gulp-iconfont#options) options.

#### Recommended options

```javascript

const RUN_TIMESTAMP = Math.round(Date.now() / 1000)
const iconsPluginOptions = {
  fontName: 'icons',
  timestamp: RUN_TIMESTAMP,
  normalize: true,
  formats: ['ttf', 'eot', 'woff', 'svg']
}

```

### Loader

#### `template`

The template option of the loader is the template for the module generated by the loader. By default the template is:

```javascript
module.export = __ICON__;
```

where `__ICON__` is an object that has the properties `fontName` (the name of the generated font, passed in the plugin options) and `text` (a string representation of the character of icon in the font).

This allows you to for example export a React element instead:

```javascript
const iconModuleTemplate = encodeURIComponent('module.exports = require("react").createElement("span", { className: "icon" }, __ICON__.text);');

const webpackConfig = {
  loaders: [{
      test: /\.svg$/,
      loader: "icons-loader?template=" + iconModuleTemplate,
  }],
  ...
}
```
