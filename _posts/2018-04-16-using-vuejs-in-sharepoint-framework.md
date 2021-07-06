---
layout: post
title: 'Using Vue.js in SharePoint Framework Applications. Part I: Whats and Whys'
date: '2018-04-16T22:15:00.000-07:00'
author: Alex Terentiev
tags:
  - SharePoint Online
  - SPFx
  - Vuejs
  - TypeScript
  - Vue
  - O365
  - SharePoint Framework
  - Office 365
  - SharePoint
modified_time: '2018-09-17T16:44:10.833-07:00'
blogger_id: 'tag:blogger.com,1999:blog-3066084330774405472.post-7870558935445576336'
blogger_orig_url: 'http://blog.aterentiev.com/2018/04/using-vuejs-in-sharepoint-framework.html'
slug: vue-js-spfx-whats-whys
---

This is the first post of a series where I'm planning to describe how to use Vue.js in SharePoint Framework apps.<br />In this post I want to provide a small introduction to Vue.js as well as describe the next parts and what to expect.<br />List of posts:<br /><ol><li>Whats and Whys (this post)</li><li><a href="/vue-js-spfx-default-web-part" target="_blank">Default SPFx web part using Vue.js</a></li><li><a href="/vue-js-spfx-yeoman-generator" target="_blank">Yeoman Generator with Vue Support</a></li><li><a href="/vue-js-spfx-prop-pane-control">Web Part Property Pane Control</a></li><li><a href="/vue-js-spfx-react-in-vue-js-solution" target="_blank">Use React Components inside Vue.js Solution</a></li></ol><a name='more'></a><h2>Why did I decide to write these posts?</h2>There are several reasons why this topic may be interesting for the community.<br />First of all, Vue.js is very popular. It means that many front-end developers prefer to use it in their projects. And if they need to implement SPFx solution they'll use Vue.js there as well.<br />The second reason is that I've heard that SharePoint community is also interested in Vue.js as an additional instrument in the toolset.<br />And the last one is my own interest in Vue.js. And it's much easier to learn something if you have some aims or commitments. <h2>Why multiple posts instead of one or two?</h2>As Vue.js is not included by SharePoint Dev team as one of OOTB templates for the projects, there probably will be a lot of things to do to make Vue.js work with SPFx, React, and Office UI Fabric controls.<br />Moreover, these posts should be some kind of detailed guide on how to create SPFx project using SPFx. And it wouldn't have happened with the single post. <h2>What is Vue.js</h2>Vue (pronounced /vjuː/, like view) is a progressive framework for building user interfaces. Unlike other monolithic frameworks, Vue is designed from the ground up to be incrementally adoptable. The core library is focused on the view layer only, and is easy to pick up and integrate with other libraries or existing projects. On the other hand, Vue is also perfectly capable of powering sophisticated Single-Page Applications when used in combination with <a href="https://vuejs.org/v2/guide/single-file-components.html" target="_blank">modern tooling</a> and <a href="https://github.com/vuejs/awesome-vue#components--libraries" target="_blank">supporting libraries</a>.<br />To learn Vue.js in details please visit the official web site of the project: <a href="https://vuejs.org/" target="_blank">https://vuejs.org/</a><h2>How to start: Vue.js, TypeScript and Node.js</h2>TypeScript is a default language of the SharePoint Framework. And Node.js is basically the core engine for modern web application.<br />As the first 2 posts we're talking about Vue.js itself, without SharePoint Framework. But it's better to use the same toolchain (or its part) for the start to be familiar with what's going on under the hood of SPFx.<br />The best way to start is to use step-by-step guidance for <a href="https://github.com/Microsoft/TypeScript-Vue-Starter" target="_blank">TypeScript-Vue-Starter</a> repo. It describes how to init the project, install needed modules, configure settings, etc. I will briefly describe the steps here as well as there are some differences in configurations between the TypeScript-Vue-Starter and SPFx projects. <h3>1. Init the Project</h3>First, let's create a basic folder structure for the project.<br />In TypeScript projects source (.ts) files are usually stored in <span class="code">src</span> folder and compiled (.js) files are stored in <span class="code">dist</span> folder.<br />Also, let's create <span class="code">components</span> subfolder in <span class="code">src</span> folder to contain the code for our components.<br />The initial folder structure will look like the one on the image below:<br /><img border="0" src="{{site.baseurl}}/assets/images/posts/2018/initial-folder-structure.png" /><br />The next step is to initialize the project using npm. It is done with the next command: 
<div markdown="1">
{% highlight console %}

npm init

{% endhighlight %}
</div>
You'll be given a series of prompts. You can use the defaults except for your entry point. You can always go back and change these in the <span class="code">package.json</span> file that's been generated for you. <h3>2. Install Dependencies</h3>We need to install TypeScript, Webpack and Vue to our project.<br />It can be done with the next command: 
<div markdown="1">
{% highlight console %}

npm i --save-dev typescript webpack webpack-cli
npm i --save vue

{% endhighlight %}
</div>
Webpack is an optional element here. For this project where we'll have a single TypeScript file we don't need a bundler.<br />But in real projects Webpack might be helpful... And it is used in SharePoint Framework, that's why I decided to use it in simple sample as well. <h3>3. Add TypeScript Configuration</h3>As we're going to use Vuejs with SharePoint Framework, let's use the same TypeScript settings as used in SPFx projects. Create a <span class="code">tsconfig.json</span> file in the root of the project with the next content: 
<div markdown="1">
{% highlight javascript %}

{
  "compilerOptions": {
    "outDir": "./built", // this property is absent in SPFx projects
    "target": "es5",
    "forceConsistentCasingInFileNames": true,
    "module": "commonjs",
    "jsx": "react",
    "declaration": true,
    "sourceMap": true,
    //"experimentalDecorators": true,
    "skipLibCheck": true,
    //"typeRoots": [
    //  "./node_modules/@types",
    //  "./node_modules/@microsoft"
    //],
    "types": [
      "es6-promise",
      "webpack-env"
    ],
    "lib": [
      "es5",
      "dom",
      "es2015.collection"
    ],
    "include": [
        "./src/**/*"
    ]
  }
}

{% endhighlight %}
</div>
  <h3>4. Configure Webpack</h3>Now we need to add <span class="code">webpack.config.js</span> file to bundle the app. As I mentioned above, Webpack is optional for this project. So, you can try to create your one without it at all. 
<div markdown="1">
{% highlight javascript %}

var path = require('path')
var webpack = require('webpack')

module.exports = {
  entry: './src/index.ts',
  output: {
    path: path.resolve(__dirname, './dist'),
    publicPath: '/dist/',
    filename: 'build.js'
  },
  module: {
    rules: [
      {
        test: /\.(png|jpg|gif|svg)$/,
        loader: 'file-loader',
        options: {
          name: '[name].[ext]?[hash]'
        }
      }
    ]
  },
  resolve: {
    extensions: ['.ts', '.js', '.vue', '.json'],
    alias: {
      'vue$': 'vue/dist/vue.esm.js'
    }
  },
  devServer: {
    historyApiFallback: true,
    noInfo: true
  },
  performance: {
    hints: false
  },
  devtool: '#eval-source-map'
}

if (process.env.NODE_ENV === 'production') {
  module.exports.devtool = '#source-map'
  // http://vue-loader.vuejs.org/en/workflow/production.html
  module.exports.plugins = (module.exports.plugins || []).concat([
    new webpack.DefinePlugin({
      'process.env': {
        NODE_ENV: '"production"'
      }
    }),
    new webpack.optimize.UglifyJsPlugin({
      sourceMap: true,
      compress: {
        warnings: false
      }
    }),
    new webpack.LoaderOptionsPlugin({
      minimize: true
    })
  ])
}

{% endhighlight %}
</div>
<h3>5. Add Build script in Node config</h3>Node allows to include "scripts" into <span class="code">package.json</span> to list a set of actions that can be run from command prompt using node engine.<br />Our aim is to add "build" script that will run Webpack to bundle the project.<br />To do that add 
<div markdown="1">
{% highlight javascript %}

"build": "webpack"

{% endhighlight %}
</div>
to the <span class="code">scripts</span> section of the <span class="code">package.json</span> file.<br />It should look like that: 
<div markdown="1">
{% highlight javascript %}

"scripts": {
    "build": "webpack",
    "test": "test"
}

{% endhighlight %}
</div>
After the script is added we can be able to call webpack and build the project from command line using next command: 
<div markdown="1">
{% highlight console %}

npm run build

{% endhighlight %}
</div>
<h3>6. Basic Project</h3>Let's create a simple Vue component using TypeScript.<br />First, create <span class="code">index.ts</span> file in <span class="code">src</span> folder and add the next content: 
<div markdown="1">
{% highlight typescript %}

import Vue from "vue";

let v = new Vue({
    el: "#app",
    template: `
    <div>
        <div>Hello {{ "{{" }}name}}!</div>
        Name: <input v-model="name" type="text">
    </div>`,
    data: {
        name: "World"
    }
});

{% endhighlight %}
</div>
It's a simple Vue component that displays a label based on the value entered to the input.<br />Now let's create <span class="code">index.html</span> in the root of the folder to test our Vue component: 
<div markdown="1">
{% highlight html %}

<!doctype html>
<html>
<head></head>

<body>
    <div id="app"></div>
</body>
<script src="./dist/build.js"></script>

</html>

{% endhighlight %}
</div>
Now let's run <span class="code">npm run build</span> and open <span class="code">index.html</span> in the browser.<br />The result should be similar to the one on the image below:<br /><img border="0" src="{{site.baseurl}}/assets/images/posts/2018/vue-basic.png" /><br /><br />Congrats! Now we have our first project implemented using Vue and TypeScript. <h2>Conclusion</h2>During this post I described what's Vue.js and how it can be used with TypeScript.<br />Now we're closer to adding Vue to the SharePoint Framework solutions.<br />The code for this example is available <a href="https://github.com/AJIXuMuK/vuejs/tree/master/basics" target="_blank">here</a><br />In the next post I will show how to implement basic web part (the one that is included in the SPFx template) using Vue.<br />That's it for now!<br />Have fun!
