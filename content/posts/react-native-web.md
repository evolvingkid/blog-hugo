---
title: "React Native Web"
Summary: "Configuring react native web on an existing react native project. (The proper way)"
date: 2023-08-05T16:56:09+05:30
---

Currently, most companies are choosing a hybrid system for their app and web development. The common framework that people go for is either flutter or react. Unlike Flutter which has a single code base for both web IOS and Android. React has 2 different for web and mobile devices. Some said it is good as Flutter because anyone who knows React can start to code in React Native which is more or less correct. But it doesn't solve the basic issues where you need to code for two different devices.

> 😂 The next paragraph talks about the company I work for and why we go for react native on the web. 🥲 (Best if you are trying to decide implement React Native Web)

I’m working on a fin-tech startup in India named Tortoise. For the past year, we were a C2B company where we have an application for customers to create a saving plan. This application had gone through multiple iterations and fixes on both UI and functionality. Like any other early-stage startup we pivoted to B2B from C2B. This means that the tech side needs to change into something like SASS company. We need to make a web application just like our app which looks with the same UI and the same logic for our customer. It took one month for completing the functions in our app and another month to perfect the UI. So doing the same on the web with React JS project from scratch will kill a lot of our time. Also, we need more and more developers to handle both projects. Which is again not good for startups. You may think we can create React JS applications and share all our hooks, redux and other logic as a package. This method is good but if we need to add logic or fix issues we need to go into the package then build it and use it on the application. There is also extra work because we don't have a separate package for handling UI logic (We do have a private package for handling connection with servers) and we need to make all the components again for the web.

# React Native Web

As a said before we have an application that works better so if we can somehow support the web with the same code base with minimal code changes then it will be a lifesaver. 

This is where [React Native Web](https://necolas.github.io/react-native-web/) comes into play. We connected React Native Web to our existing react native app and keep the structure as same. We don’t use mono-repo or shared packages for all components and logic in the app because as I said in the previous paragraph it's too much extra work.

# React Native Web on existing React Native code base

In this section, I will be sharing for to set up react native web on your current react native app and all the issues that will be coming up with third-party packages on your existing code.

All our logic is in a services kind of function instead of used directly on the UI this also helps us to make everything support the web quickly.

# React Native Web setup

First, you need to install React native web package:

```shell
npm install react-dom react-native-web
```

The”react-native-web” package will be providing us with react native equivalent web supported function/components.

We will be using Webpack to run the web application: 

```shell
npm install -dev babel-plugin-react-native-web webpack webpack-cli webpack-dev-server html-webpack-plugin react-dom babel-loader url-loader @svgr/webpack 
```

I will be listing down the package version we are using so that any error with a specific package version can be avoided.

```json
{
"react-native": "0.68.5",
"react-dom": "^17.0.2",
"react": "17.0.2",
"babel-plugin-react-native-web": "^0.18.10",
"webpack": "^5.75.0",
"webpack-cli": "^5.0.1",
"webpack-dev-server": "^4.11.1",
"html-webpack-plugin": "^5.5.0",
"babel-loader": "^9.1.2",
"url-loader": "^4.1.1",
"@svgr/webpack": "^6.5.1",
}
```

Like the Android and IOS folders in our React native project, we will make a folder on the root named “web”.  Just like Android and IOS, the web will have all the web-specific configs (Something like a public folder in React Js).

```shell
mkdir web
```

create an “index.html” file inside the web folder and paste the below HTML code:

```html
<!DOCTYPE html>
<html>

<head>
  <meta charset="UTF-8" />

  <link rel="icon" type="image/x-icon" href="./favicon.ico">

  <meta name="viewport" content="width=device-width, initial-scale=1" />
  <meta name="theme-color" content="#000000" />
  <meta name="viewport" content="width=device-width, initial-scale=1.0, maximum-scale=1.0,user-scalable=0" />
  <meta http-equiv="X-UA-Compatible" content="ie=edge" />


  <title>Tortoise</title>

  <style>
    html {
      height: 100%;
    }

    body {
      height: 100%;
    }

    #app-root {
      display: flex;
      flex: 1;
      height: 100%;
    }
  </style>
</head>

<body>
  <noscript>You need to enable JavaScript to run this app.</noscript>
  <div id="app-root"></div>
</body>

</html>
```

Don’t change the div id “app-root”. Because that is where React will inject the web code into your HTML. You can add a favicon in the same web folder.

Create an “index.web.js” file at the root of your React native project. This is the starting point for your web app-like “index.js” file inside the src folder in React JS. 

> We will be using “file_name.web.file_extension“ as the web support file of the react “file_name.file_extension”. For example, we have an “index.js” file on the root of your app for where the react native starts. For that, we have an “index.web.js” file on the root of your application which has the same function as a “index.js” but for the web only. Whenever you use the “.web” file the webpack will use that file as the web-supported file.

Paste the below code in your “index.web.js”  file at the root of your project.

```jsx
import React from 'react';
import {AppRegistry} from 'react-native';
import {name as appName} from './app.json';
import App from './src/App.web'

AppRegistry.registerComponent(appName, () => App);
AppRegistry.runApplication(appName, {
  initialProps: {},
  rootTag: document.getElementById('app-root'),
});
```

Create a “webpack.config.js” at the root of your application. and paste the code below.

```js
const path = require('path');

const webpack = require('webpack');
const HtmlWebpackPlugin = require('html-webpack-plugin');

const appDirectory = path.resolve(__dirname);
const { presets } = require(`${appDirectory}/babel.config.js`);

const compileNodeModules = [
  'react-native-gesture-handler',
].map(moduleName => path.resolve(appDirectory, `node_modules/${moduleName}`));

const babelLoaderConfiguration = {
  test: /\.js$|tsx?$/,
  // Add every directory that needs to be compiled by Babel during the build.
  include: [
    path.resolve(__dirname, 'index.web.js'), // Entry to your application
    path.resolve(__dirname, 'src/App.web.tsx'), // Change this to your main App file
    path.resolve(__dirname, 'src'),
    path.resolve(__dirname, 'assets'),
    ...compileNodeModules,
  ],
  use: {
    loader: 'babel-loader',
    options: {
      cacheDirectory: true,
      presets,
      plugins: ['react-native-web'],
    },
  },
};

const svgLoaderConfiguration = {
  test: /\.svg$/,
  use: [
    {
      loader: '@svgr/webpack',
    },
  ],
};


const imageLoaderConfiguration = {
  test: /\.(png|jpg|gif|jpe?g|ico)$/,
  use: {
    loader: 'url-loader',
    options: {
      name: '[name].[ext]',
    },
  },
};

const cssLoaderConfig = {
  test: /\.css$/,
  use: ['style-loader', 'css-loader'],
};


module.exports = {
  entry: {
    app: [path.join(__dirname, 'index.web.js'), '@babel/polyfill'],
  },
  output: {
    path: path.resolve(appDirectory, 'dist'),
    publicPath: '/',
    filename: 'rnw_tortoise.bundle.js',
  },
  devServer: {
    historyApiFallback: true
  },
  resolve: {
    extensions: ['.web.tsx', '.web.ts', '.tsx', '.ts', '.web.js', '.js', '.css'],
    alias: {
      'react-native$': 'react-native-web',
    },
  },
  module: {
    rules: [
      babelLoaderConfiguration,
      imageLoaderConfiguration,
      svgLoaderConfiguration,
      cssLoaderConfig,
    ],
  },
  plugins: [
    new webpack.ProvidePlugin({
      React: 'react',
    }),
    new HtmlWebpackPlugin({
      template: path.join(__dirname, 'web/index.html'),
      favicon: path.join(__dirname, 'web/favicon.ico'),
    }),
    new webpack.HotModuleReplacementPlugin(),
    new webpack.DefinePlugin({
      __DEV__: JSON.stringify(true),
    }),
    new webpack.EnvironmentPlugin({ JEST_WORKER_ID: null }),
    new webpack.DefinePlugin({ process: { env: {} } })
  ],
};
```

> Now I will explain all the important stuff going on with webpack config. Since most of the issues that you may get on react native applications on the web can be simple fixes from the webpack.

```js
entry: {
    app: [path.join(__dirname, 'index.web.js'), '@babel/polyfill'],
  },
```

This defines where your code starts to execute. As I explain before create “index.web.js” which will start the execution.

```js
 output: {
    path: path.resolve(appDirectory, 'dist'),
    publicPath: '/',
    filename: 'rnw_tortoise.bundle.js',
  },
```

This is the bundle output that will run your application. If you needed to change the bundle file name you can change the filename object. Currently, I set it to “rnw_tortoise”.

```js
 resolve: {
    extensions: ['.web.tsx', '.web.ts', '.tsx', '.ts', '.web.js', '.js', '.css'],
    alias: {
      'react-native$': 'react-native-web',
    },
  },
```

In the resolve object, it has an “extensions” array which marks all the file types that webpack resolves. I had tsx and ts extensions because we use typescript in our React native app. If your project doesn’t use typescript you can remove it. No error will happen even if you didn’t remove it so best to keep it that way if you are not sure

Alias object is one of the important things. As you know react native is out of the box don’t support the web. But react-native-web supports the web. So what the alias does is that it will map your react-native import in your code to react-native-web when webpack is running. Alias is the main thing while using react native web. There are multiple packages that don't support the web like react-native-Lottie and fast-image so you can use “alias” and map it to another package that supports the web.

> Don’t worry if you are not able to understand it's working. We will be using it for other third-party packages as we go on. 

```js
const compileNodeModules = [
  'react-native-gesture-handler',
].map(moduleName => path.resolve(appDirectory, `node_modules/${moduleName}`));

const babelLoaderConfiguration = {
  test: /\.js$|tsx?$/,
  // Add every directory that needs to be compiled by Babel during the build.
  include: [
    path.resolve(__dirname, 'index.web.js'), // Entry to your application
    path.resolve(__dirname, 'src/App.web.tsx'), // Change this to your main App file
    path.resolve(__dirname, 'src'),
    path.resolve(__dirname, 'assets'),
    ...compileNodeModules,
  ],
  use: {
    loader: 'babel-loader',
    options: {
      cacheDirectory: true,
      presets,
      plugins: ['react-native-web'],
    },
  },
};

const svgLoaderConfiguration = {
  test: /\.svg$/,
  use: [
    {
      loader: '@svgr/webpack',
    },
  ],
};


const imageLoaderConfiguration = {
  test: /\.(png|jpg|gif|jpe?g|ico)$/,
  use: {
    loader: 'url-loader',
    options: {
      name: '[name].[ext]',
    },
  },
};

const cssLoaderConfig = {
  test: /\.css$/,
  use: ['style-loader', 'css-loader'],
};


// the module object

 module: {
    rules: [
      babelLoaderConfiguration,
      imageLoaderConfiguration,
      svgLoaderConfiguration,
      cssLoaderConfig,
    ],
  },
```
We are using webpack and the webpack cant compile CSS, SVG, SVG and JSX. It can only compile JS out of the box. So we need to set the loaders which will convert these files into something JS code that webpack can work on.

babelLoaderConfiguration handles all our JSX code and JS code in our project. As you can see there is an object name “include” in babelLoaderConfiguration. Connect says to include the folder/file that I want to include in webpack babelLoaderConfiguration. If you have any JSX or JS code that is outside your src folder then please add it to the babelLoaderConfiguration.

compileNodeModules modules handle all the packages. For example, we use pan-gesture and flatlist imported from react-native-gesture-handler. As you know it won't support react-native-web so if you need that react-native-package to support your project you need to add them or else you will get an error saying “can’t resole need to add loader” from webpack. Please note that it will only work on a package that is inside your “nodemodule” folder.

imageLoaderConfiguration is just the same as babelLoaderConfiguration where imageLoaderConfiguration handles the image file and covert it in a way that webpack will understand. That same goes for cssLoaderConfig.

svgLoaderConfiguration module same as cssLoaderConfig. webpack doesn’t understand SVG so needs a loader to handle it. We use @svgr/webpack in svgLoaderConfiguration because we were using “react-native-svg” package to use SVG in our project as you know with this we can use SVG as a normal JSX component. So we also need the same feature on our web or else we have to spend another 3 days to change all the SVG specific to the web. (Mostly because we use so many local SVG).

```js

plugins: [
    new webpack.ProvidePlugin({
      React: 'react',
    }),
    new HtmlWebpackPlugin({
      template: path.join(__dirname, 'web/index.html'),
      favicon: path.join(__dirname, 'web/favicon.ico'),
    }),
    new webpack.HotModuleReplacementPlugin(),
    new webpack.DefinePlugin({
      __DEV__: JSON.stringify(true),
    }),
    new webpack.EnvironmentPlugin({ JEST_WORKER_ID: null }),
    new webpack.DefinePlugin({ process: { env: {} } })
  ],

```

In the plugin section, you may only want to look at a few things. The first is “HtmlWebpackPlugin”. As you can see the template is pointed to “web/index.html”. That is why in the above step we mentioned making a “web” folder and creating an “index.html“inside the web folder. If you want to change the web folder to the public or something else change the path name here too.

In DefinePlugin allow us to set a global variable. For example “__DEV__” is something that react-native has but React JS doesn’t have it. without it, your application will break whenever it encounters “__DEV__”. Please change it to false when you are building or running a production env. If you are using “__DEV__” as envs.

Now create an “App.web.tsx” inside the src folder. Copy and paste the below code:


```jsx
....
const App = () => {
  return (
    <View >
      <Text>Welcome to react-native-web</Text>
    </View>
  );
};

export default App;
```

Add these scripts into that package.json file:

```js
"web": "webpack serve --mode=development --config webpack.config.js",
"web-build": "rm -rf dist/ && webpack --mode=production --config webpack.config.js"
```

Now you can run the web using “npm run web” and build it using “npm run web-build”.


# Third-party packages in React Native

Most of the issues of React native web will be using third-party-package. There are a few workaround and fixes you can do it solve this issue. 

For all the UI-related packages (packages that don't use native UI) there is a quick fix to add support for the package to react-native-web. You can add the package name to the 'compileNodeModules' array and restart the webpack.

For all other third-party packages, there are 2 ways to solve the issues.

**Method 1**: Using the counter web support package. You can install the counter web support package and use 'alias' to use those packages on the web.

```js
alias: {
    'react-native$': 'react-native-web',
	'<react-native-package>': '<counter-web-supported-package>'
},

```

**Method 2**: You can create your package/component. Save it as a web.tsx/web.ts file alongside with currently used package.

```js

components
|
|--myComponent.tsx
|--myComponent.web.tsx

// on the web it will choose the web.tsx file and 
// on mobile devices it will choose tsx

```