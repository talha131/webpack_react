# Developing with Webpack

If you are not one of those people who likes to skip the introductions, you might have some clue what Webpack is. In its simplicity, it is a module bundler. It takes a bunch of assets in and outputs assets you can give to your client.

This sounds simple, but in practice, it can be a complicated and messy process. You definitely don't want to deal with all the details yourself. This is where Webpack fits in. Next, we'll get Webpack set up and your first project running in development mode.

W> Before getting started, make sure you are using a recent version of Node.js as that will save some trouble. There are [packages available for many platforms](https://nodejs.org/en/download/package-manager/). A good alternative is to set up a [Vagrant](https://www.vagrantup.com/) box and maintain your development environment there.

T> Especially *css-loader* has [issues with Node 0.10](https://github.com/webpack/css-loader/issues/144) given it's missing native support for promises. Consider polyfilling `Promise` through `require('es6-promise').polyfill()` at the beginning of your Webpack configuration if you still want to use 0.10. This technique depends on [es6-promise](https://www.npmjs.com/package/es6-promise) package.

## Setting Up the Project

Webpack is one of those tools that depends on [Node.js](http://nodejs.org/). Make sure you have it installed and that you have `npm` available at your terminal. Set up a directory for your project, navigate there, execute `npm init`, and fill in some details. You can just hit *return* for each and it will work. Here are the commands:

```bash
mkdir kanban_app
cd kanban_app
npm init -y # -y gives you default *package.json*, skip for more control
```

As a result, you should have *package.json* at your project root. You can still tweak it manually to make further changes. We'll be doing some changes through *npm* tool, but it's fine to tweak the file to your liking. The official documentation explains various [package.json options](https://docs.npmjs.com/files/package.json) in more detail. I also cover some useful library authoring related tricks later in this book.

T> You can set those `npm init` defaults at *~/.npmrc*. See the "Authoring Libraries" for more information about npm and its usage.

### Setting Up Git

If you are into version control, as you should, this would be a good time to set up your repository. You can create commits as you progress with the project.

If you are using git, I recommend setting up a *.gitignore* to the project root:

**.gitignore**

```bash
node_modules
```

At the very least, you should have *node_modules* here as you probably don't want that to end up in the source control. The problem with that is that as some modules need to be compiled per platform, it gets rather messy to collaborate. Ideally, your `git status` should look clean. You can extend *.gitignore* as you go.

T> You can push operating system level ignore rules, such as *.DS_Store* and *\*.log* to *~/.gitignore*. This will keep your project level rules simpler.

## Installing Webpack

Next, you should get Webpack installed. We'll do a local install and save it as a project dependency. This will allow us to maintain Webpack's version per project. Execute

```bash
npm i webpack --save-dev
```

npm maintains a directory where it installs possible executables of packages. You can display the exact path using `npm bin`. Most likely it points at `.../node_modules/.bin`. Try triggering Webpack from there using `node_modules/.bin/webpack` or a similar command.

You should see a version log, a link to the command line interface guide and a long list of options. We won't be using most of those, but it's good to know that this tool is packed with functionality, if nothing else.

Webpack works using a global install as well (`-g` or `--global` flag during installation). It is preferred to keep it as a project dependency instead. This way you have direct control over the version you are running.

We will be using `--save` and `--save-dev` to separate application and development dependencies. The separation keeps project dependencies more understandable. This will come in handy when we generate a vendor bundle later on.

T> There are handy shortcuts for `--save` and `--save-dev`. `-S` maps to `--save` and `-D` to `--save-dev`. So if you want to optimize for characters written, consider using these instead.

## Directory Structure

As projects with just *package.json* are boring, we should set up something more concrete. To get started, we can implement a little web site that loads some JavaScript which we then build using Webpack. Set up a structure like this:

- /app
  - index.js
  - component.js
- /build
  - index.html
- package.json
- webpack.config.js

In this case, we'll generate *bundle.js* using Webpack based on our */app*. To make this possible, we should set up some assets and *webpack.config.js*.

## Setting Up Assets

As you never get tired of `Hello world`, we might as well model a variant of that. Set up a component like this:

**app/component.js**

```javascript
module.exports = function () {
  var element = document.createElement('h1');

  element.innerHTML = 'Hello world';

  return element;
};
```

Next, we are going to need an entry point for our application. It will simply `require` our component and render it through the DOM:

**app/index.js**

```javascript
var component = require('./component');
var app = document.createElement('div');

document.body.appendChild(app);

app.appendChild(component());
```

We are also going to need some HTML so we can load the bundle:

**build/index.html**

```html
<!DOCTYPE html>
<html lang="en">
  <head>
    <meta charset="utf-8">
    <title>Kanban app</title>
  </head>
  <body>
    <div id="app"></div>

    <script src="./bundle.js"></script>
  </body>
</html>
```

We'll generate this dynamically at the build chapter, but this is enough for now.

## Setting Up Webpack Configuration

We'll need to tell Webpack how to deal with the assets we just set up. For this purpose we'll build *webpack.config.js*. Webpack and its development server will be able to discover this file through convention.

To map our application to *build/bundle.js* we need configuration like this:

**webpack.config.js**

```javascript
const path = require('path');

const PATHS = {
  app: path.join(__dirname, 'app'),
  build: path.join(__dirname, 'build')
};

module.exports = {
  // Entry accepts a path or an object of entries.
  // The build chapter contains an example of the latter.
  entry: PATHS.app,
  output: {
    path: PATHS.build,
    filename: 'bundle.js'
  }
};
```

The `entry` path could be given as a relative one. The [context](https://webpack.github.io/docs/configuration.html#context) field can be used to configure that lookup. Given plenty of places expect absolute paths, I prefer to use absolute paths everywhere to avoid confusion.

I like to use `path.join`, but `path.resolve` would be a good alternative. `path.resolve` is equivalent to navigating the file system through *cd*. `path.join` gives you just that, a join. See [Node.js path API](https://nodejs.org/api/path.html) for the exact details.

If you trigger `node_modules/.bin/webpack`, you should see output like this:

```bash
Hash: e7dd4bf8a294b5c93141
Version: webpack 1.12.11
Time: 734ms
    Asset   Size  Chunks             Chunk Names
bundle.js  12 kB       0  [emitted]  main
   [0] ./app/index.js 169 bytes {0} [built]
   [5] ./app/component.js 136 bytes {0} [built]
    + 4 hidden modules
```

This means you have a build at your output directory. You can open the `build/index.html` file directly through a browser to examine the results. On OS X `open ./build/index.html` works.

T> Another way to serve the contents of the directory through a server, such as *serve* (`npm i serve -g`). In this case, execute `serve` at the output directory and head to `localhost:3000` at your browser. You can configure the port through the `--port` parameter.

## Adding a Build Shortcut

Given executing `node_modules/.bin/webpack` is a little verbose, we should do something about it. npm and *package.json* double as a task runner with some configuration. Adjust it as follows:

**package.json**

```json
...
"scripts": {
  "build": "webpack"
},
...
```

You can execute the scripts defined this way through *npm run*. If you trigger *npm run build* now, you should get a build at your output directory like earlier.

This works because npm adds `node_modules/.bin` temporarily to the path. As a result, rather than having to write `"build": "node_modules/.bin/webpack"`, we can do just `"build": "webpack"`. Unless Webpack is installed to the project, this can point to a possible global install. That can be potentially dangerous as it's a good idea to have control over the version of tools you are using.

The scheme can be expanded further. Task runners, such as Grunt or Gulp, allow you to achieve the same while operating in a cross-platform manner. If you go through *package.json* like this, you may have to be more careful. On the plus side, this is a very light approach. To keep things simple, we'll be relying on it.

## Setting Up *webpack-dev-server*

As developing your application through a build script like this will get boring eventually, Webpack provides neater means for development in particular. *webpack-dev-server* is a development server running in-memory. It refreshes content automatically in the browser while you develop your application. This makes it roughly equivalent to tools, such as [LiveReload](http://livereload.com/) or [Browsersync](http://www.browsersync.io/).

The greatest advantage Webpack has over these tools is Hot Module Replacement (HMR). In short, it provides a way to patch the browser state without a full refresh. We'll discuss it in more detail when we go through the React setup.

W> You should use *webpack-dev-server* strictly for development. If you want to host your application, consider other, standard solutions, such as Apache or Nginx.

To get started with *webpack-dev-server*, execute

```bash
npm i webpack-dev-server --save-dev
```

at the project root to get the server installed.

Just like above, we'll need to define an entry point to the `scripts` section of *package.json*. Given our *index.html* is below *./build*, we should let *webpack-dev-server* to serve the content from there. We'll move this to Webpack configuration later, but this will do for now:

**package.json**

```json
...
"scripts": {
leanpub-start-delete
  "build": "webpack"
leanpub-end-delete
leanpub-start-insert
  "build": "webpack",
  "start": "webpack-dev-server --content-base build"
leanpub-end-insert
},
...
```

If you trigger either *npm run start* or *npm start* now, you should see something like this at the terminal:

```bash
> webpack-dev-server

http://localhost:8080/
webpack result is served from /
content is served from .../kanban_app/build
404s will fallback to /index.html

webpack: bundle is now VALID.
```

The output means that the development server is running. If you open *http://localhost:8080/* at your browser, you should see something. If you try modifying the code, you should see output at your terminal. The problem is that the browser doesn't catch these changes without a hard refresh. That's something we need to resolve next.

![Hello world](images/hello_01.png)

T> If you fail to see anything at the browser, you may need to use a different port through *webpack-dev-server --port 3000* kind of invocation. One reason why the server might fail to run is simply because there's something else running in the port. You can verify this through a terminal command, such as `netstat -na | grep 8080`. If there's something running in the port 8080, it should display a message. The exact command may depend on your platform.

### Splitting Up Configuration

As the development setup has certain requirements of its own, we'll need to split our Webpack configuration. Given Webpack configuration is just JavaScript, there are many ways to achieve this. At least the following ways are feasible:

* Maintain configuration in multiple files and point Webpack to each through `--config` parameter. Share configuration through module imports. You can see this approach in action at [webpack/react-starter](https://github.com/webpack/react-starter).
* Push configuration to a library which you then consume. Example: [HenrikJoreteg/hjs-webpack](https://github.com/HenrikJoreteg/hjs-webpack).
* Maintain configuration within a single file and branch there. If we trigger a script through *npm* (i.e., `npm run test`), npm sets this information in an environment variable. We can match against it and return the configuration we want.

I prefer the last approach as it allows me to understand what's going on easily. It is ideal for small projects, such as this.

To keep things simple and help with the approach, I've defined a custom `merge` function that concatenates arrays and merges objects. This is convenient with Webpack as we'll soon see. Execute

```bash
npm i webpack-merge --save-dev
```

to add it to the project.

Next, we need to define some split points to our configuration so we can customize it per npm script. Here's the basic idea:

**webpack.config.js**

```javascript
...
leanpub-start-insert
const merge = require('webpack-merge');
leanpub-end-insert

leanpub-start-insert
const TARGET = process.env.npm_lifecycle_event;
leanpub-end-insert
const PATHS = {
  app: path.join(__dirname, 'app'),
  build: path.join(__dirname, 'build')
};

leanpub-start-delete
module.exports = {
leanpub-end-delete
leanpub-start-insert
const common = {
leanpub-end-insert
  // Entry accepts a path or an object of entries.
  // The build chapter contains an example of the latter.
  entry: PATHS.app,
  output: {
    path: PATHS.build,
    filename: 'bundle.js'
  }
};

leanpub-start-insert
// Default configuration
if(TARGET === 'start' || !TARGET) {
  module.exports = merge(common, {});
}

if(TARGET === 'build') {
  module.exports = merge(common, {});
}
leanpub-end-insert
```

Now that we have room for expansion, we can hook up Hot Module Replacement to make the browser refresh and make the development mode more useful.

### Configuring Hot Module Replacement (HMR)

Hot Module Replacement gives us simple means to refresh the browser automatically as we make changes. The idea is that if we change our *app/component.js*, the browser will refresh itself. The same goes for possible CSS changes.

In order to make this work, we'll need to connect the generated bundle running in-memory to the development server. Webpack uses WebSocket based communication to achieve this. To keep things simple, we'll let Webpack generate the client portion for us through the development server *inline* option. The option will include the client side scripts needed by HMR to the bundle that Webpack generates.

Beyond this we'll need to enable `HotModuleReplacementPlugin` to make the setup work. In addition I am going to enable HTML5 History API fallback as that is convenient default to have especially if you are dealing with advanced routing. Here's the setup:

**webpack.config.js**

```javascript
...
leanpub-start-insert
const webpack = require('webpack');
leanpub-end-insert

...

if(TARGET === 'start' || !TARGET) {
leanpub-start-delete
  module.exports = merge(common, {});
leanpub-end-delete
leanpub-start-insert
  module.exports = merge(common, {
    devServer: {
      contentBase: PATHS.build,

      // Enable history API fallback so HTML5 History API based
      // routing works. This is a good default that will come
      // in handy in more complicated setups.
      historyApiFallback: true,
      hot: true,
      inline: true,
      progress: true,

      // Display only errors to reduce the amount of output.
      stats: 'errors-only',

      // Parse host and port from env so this is easy to customize.
      host: process.env.HOST,
      port: process.env.PORT
    },
    plugins: [
      new webpack.HotModuleReplacementPlugin()
    ]
  });
leanpub-end-insert
}

...
```

Given we pushed `contentBase` configuration to JavaScript, we can remove it from *package.json*:

**package.json**

```json
...
"scripts": {
  "build": "webpack"
leanpub-start-delete
  "start": "webpack-dev-server --content-base build"
leanpub-end-delete
leanpub-start-insert
  "start": "webpack-dev-server"
leanpub-end-insert
},
...
```

Execute `npm start` and surf to **localhost:8080**. Try modifying *app/component.js*. It should refresh the browser. Note that this is hard refresh in case you modify JavaScript code. CSS modifications work in a neater manner and can be applied without a refresh. In the next chapter we discuss how to achieve something similar with React. This will provide us a little better development experience.

If you using Windows and it doesn't refresh, see the following section for an alternative setup.

W> *webpack-dev-server* can be very particular about paths. If the given `include` paths don't match the system casing exactly, this can cause it to fail to work. Webpack [issue #675](https://github.com/webpack/webpack/issues/675) discusses this in more detail.

T> You should be able to access the application alternatively through **localhost:8080/webpack-dev-server/** instead of root. You can see all the files the development server is serving there.

T> If you want to default to some other port than *8080*, you can use a declaration like `port: process.env.PORT || 3000`.

### HMR on Windows

The setup may be problematic on certain versions of Windows. Instead of using `devServer` and `plugins` configuration, implement it like this:

**webpack.config.js**

```javascript
...

if(TARGET === 'start' || !TARGET) {
  module.exports = merge(common, {});
}

...
```

**package.json**

```json
...
"scripts": {
  "build": "webpack",
leanpub-start-delete
  "start": "webpack-dev-server"
leanpub-end-delete
leanpub-start-insert
  "start": "webpack-dev-server --watch-poll --inline --hot"
leanpub-end-insert
},
...
```

Given this setup polls the filesystem, it is going to be more resource intensive. It's worth giving a go if the default doesn't work, though.

T> There are more details in *webpack-dev-server* issue [#155](https://github.com/webpack/webpack-dev-server/issues/155).

### Alternative Ways to Use *webpack-dev-server*

We could have passed *webpack-dev-server* options through the command line interface (CLI). I find it clearer to manage it within Webpack configuration as that helps to keep *package.json* nice and tidy.

Alternatively, we could have set up an Express server of our own and used *webpack-dev-server* as a [middleware](https://webpack.github.io/docs/webpack-dev-middleware.html). There's also a [Node.js API](https://webpack.github.io/docs/webpack-dev-server.html#api).

T> [dotenv](https://www.npmjs.com/package/dotenv) allows you to define environment variables through a *.env* file. This can be somewhat convenient for development!

W> Note that there are [slight differences](https://github.com/webpack/webpack-dev-server/issues/106) between the CLI and the Node.js API and they may behave slightly differently at times. This is the reason why some prefer to solely use the Node.js API.

### Customizing Server `host` and `port`

It is possible to customize host and port settings through the environment in our setup (i.e., `export PORT=3000` on Unix or `SET PORT=3000` on Windows). This can be useful if you want to access your server within the same network. The default settings are enough on most platforms.

To access your server, you'll need to figure out the ip of your machine. On Unix this can be achieved using `ifconfig`. On Windows `ipconfig` can be used. An npm package, such as [node-ip](https://www.npmjs.com/package/node-ip) may come in handy as well. Especially on Windows you may need to set your `HOST` to match your ip to make it accessible.

T> If you are using an environment, such as Cloud9, you should set `HOST` to `0.0.0.0`. The default `localhost` isn't always the best option as it's available only to the local machine itself. `0.0.0.0` is available for all network interfaces.

## Refreshing CSS

We can extend this approach to work with CSS. Webpack allows us to change CSS without forcing a full refresh. To load CSS into a project, we'll need to use a couple of loaders. To get started, invoke

```bash
npm i css-loader style-loader --save-dev
```

T> If you are using Node.js 0.10, this is a good time to get a [ES6 Promise polyfill](https://github.com/jakearchibald/es6-promise#auto-polyfill) set up.

Now that we have the loaders we need, we'll need to make sure Webpack is aware of them. Configure as follows.

**webpack.config.js**

```javascript
...

const common = {
  ...
  },
leanpub-start-insert
  module: {
    loaders: [
      {
        // Test expects a RegExp! Note the slashes!
        test: /\.css$/,
        loaders: ['style', 'css'],
        // Include accepts either a path or an array of paths.
        include: PATHS.app
      }
    ]
  }
leanpub-end-insert
}

...
```

The configuration we added means that files ending with `.css` should invoke given loaders. `test` matches against a JavaScript style regular expression. The loaders are evaluated from right to left. In this case, *css-loader* gets evaluated first, then *style-loader*. *css-loader* will resolve `@import` and `url` statements in our CSS files. *style-loader* deals with `require` statements in our JavaScript. A similar approach works with CSS preprocessors, like Sass and Less, and their loaders.

T> Loaders are transformations that are applied to source files, and return the new source. Loaders can be chained together, like using a pipe in Unix. See Webpack's [What are loaders?](http://webpack.github.io/docs/using-loaders.html) and [list of loaders](http://webpack.github.io/docs/list-of-loaders.html).

W> If `include` isn't set, Webpack will traverse all files within the base directory. This can hurt performance! It is a good idea to set up `include` always. There's also `exclude` option that may come in handy. Prefer `include`, however.

We are missing just one bit, the actual CSS itself:

**app/main.css**

```css
body {
  background: cornsilk;
}
```

Also, we'll need to make Webpack aware of it. Without having a `require` pointing at it, Webpack won't be able to find the file:

**app/index.js**

```javascript
leanpub-start-insert
require('./main.css');
leanpub-end-insert

...
```

Execute `npm start` now. Point your browser to **localhost:8080** if you are using the default port.

Open up *main.css* and change the background color to something like `lime` (`background: lime`). Develop styles as needed to make it look a little nicer.

![Hello cornsilk world](images/hello_02.png)

## Enabling Sourcemaps

To improve the debuggability of the application, we can set up sourcemaps. They allow you to see exactly where an error was raised. In Webpack this is controlled through the `devtool` setting. We can use a decent default as follows:

**webpack.config.js**

```javascript
...

if(TARGET === 'start' || !TARGET) {
  module.exports = merge(common, {
leanpub-start-insert
    devtool: 'eval-source-map',
leanpub-end-insert
    ...
  });
}

...
```

If you run the development build now using `npm start`, Webpack will generate sourcemaps. Webpack provides many different ways to generate them as discussed in the [official documentation](https://webpack.github.io/docs/configuration.html#devtool). In this case, we're using `eval-source-map`. It builds slowly initially, but it provides fast rebuild speed and yields real files.

Faster development specific options, such as `cheap-module-eval-source-map` and `eval`, produce lower quality sourcemaps. Especially `eval` is fast and is the most suitable for large projects.

It is possible you may need to enable sourcemaps in your browser for this to work. See [Chrome](https://developer.chrome.com/devtools/docs/javascript-debugging) and [Firefox](https://developer.mozilla.org/en-US/docs/Tools/Debugger/How_to/Use_a_source_map) instructions for further details.

## Linting the Project

I discuss linting in detail in the *Linting in Webpack* chapter. Consider integrating that setup into your project now as that will save some debugging time. It will allow you to pick up certain categories of errors earlier.

## Conclusion

In this chapter, you learned to build and develop using Webpack. I will return to the build topic at *Building Kanban*. The current setup is not ideal. At this point it's the development configuration that matters. In the next chapter, we will see how to expand the approach to work with React.
