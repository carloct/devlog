+++
categories = ["development"]
date = "2017-05-24T04:38:17+01:00"
description = "Vuejs is great but what about React?"
tags = ["laravel", "react"]
title = "Guide on React with Laravel 5.4"

+++

Laravel 5.4 comes shipped with a nice integration with Vuejs, you install the framework and everything is already in place, there are even few example files you can start from, but what about React?

Vuejs is a *great* framework, but I like the less conventional approach of React and the more active community, and thanks to the amazing [Laravel Mix](https://laravel.com/docs/5.4/mix), switching to React is a no brainer.

#### Create a Laravel project

```
composer create-project --prefer-dist laravel/laravel laravel-react
```

And install the npm dependencies

```
npm install
```

Add react and react-dom as dependencies

```
npm install --save react react-dom
```

#### Set up Laravel Mix

Laravel 5.4 relies on Mix for the assets compiling, it's essentially a wrapper around Webpack, and it exposes a method for using React out of box.

In `webpack.min.js` change the line:

```javascript
mix.js('resources/assets/js/app.js', 'public/js')
   .sass('resources/assets/sass/app.scss', 'public/css');
```

To:

```javascript
mix.react('resources/assets/js/app.js', 'public/js')
   .sass('resources/assets/sass/app.scss', 'public/css');
```

now run `npm run dev`, Mix will download all the dependencies required to transpile React jsx.

That's it, Mix did all the hard work for us. You can even run `npm run watch` and it will recompile the javascript as soon as you save the file.

#### Clean up

Laravel comes with some references to Vue that you want to remove since you won't be using them.

In `resources/assets/js/app.js`, delete the content and replace it with:

```javascript
import React from 'react';
import ReactDom from 'react-dom';


 ReactDom.render(
   <h1>Hello, React!</h1>,
   document.getElementById('root')
 );
```

In `package.json` remove Vue from the dependencies, and delete `resources/assets/js/bootstrap.js`

#### Test it

Open `resources/views/welcome.blade.php` and add a `<script>` tag before the closing `</body>` tag

```
<script type="text/javascript" src="js/app.js"></script>
```

 then add a div that will be your root react element, anywhere within the `<body>` tag

```html
<div id="root"></div>
```

Build the js modules

```
npm run dev
```

Test it with the built-in laravel server

```
php artisan serve
```

and you should see

![Hello React](/img/hello-react.png)

If you want to take full advantage of ES6, you should also install the relative babel modules:

```
npm install --only=dev babel-preset-es2015 babel-preset-stage-2
```

and drop a `.babelrc` file in the project root folder

```javascript
{
    "presets": ["react", "es2015", "stage-2"]
}
```
