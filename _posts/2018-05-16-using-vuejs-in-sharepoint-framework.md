---
layout: post
title: 'Using Vue.js in SharePoint Framework Applications. Part II: Default SPFx web part using Vue.js'
date: '2018-05-16T22:30:00.001-07:00'
author: Alex Terentiev
tags:
    - SharePoint Online
    - Client Side Web Part
    - SPFx
    - Vuejs
    - Client Web Part
    - Vue
    - Gulp
    - O365
    - SharePoint Framework
    - Office 365
    - SharePoint
modified_time: '2018-10-07T11:58:08.955-07:00'
blogger_id: 'tag:blogger.com,1999:blog-3066084330774405472.post-2401117063856284271'
blogger_orig_url: 'http://blog.aterentiev.com/2018/05/using-vuejs-in-sharepoint-framework.html'
slug: vue-js-spfx-default-web-part
---

This is the second post about SharePoint Framework and Vue.js. In this post I'm going to implement basic Client-Side Web Part using Vue.js - basically, "wrap" the markup from Web Part template project with Vue.js  component.<br />List of posts:<br /><ol><li><a href="/vue-js-spfx-whats-whys" target="_blank">Whats and Whys</a></li><li>Default SPFx web part using Vue.js (this post)</li><li><a href="/vue-js-spfx-yeoman-generator" target="_blank"> Yeoman Generator with Vue Support</a></li><li><a href="/vue-js-spfx-prop-pane-control">Web Part Property Pane Control</a></li><li><a href="/vue-js-spfx-react-in-vue-js-solution" target="_blank">Use React Components inside Vue.js Solution</a></li></ol><a name='more'></a>In the first post we went through the basic configuration of a project to use both TypeScript and Vue.js.<br />Now let's go further and include Vue.js in SharePoint Framework project<br /><h2>1. Scaffolding</h2>So, the first thing we should do, as with any other SPFx project, is to "scaffold" it from the template how it is described <a href="https://docs.microsoft.com/en-us/sharepoint/dev/spfx/web-parts/get-started/build-a-hello-world-web-part#create-a-new-web-part-project" target="_blank">here</a><br />This will create a basic project with a markup implemented using HTML with no framework used.<br /><img border="0" src="{{site.baseurl}}/assets/images/posts/2018/html-web-part.png" /><br />Now we have a SPFx project prepared and the goal is to replace the HTML markup with a Vue component. <br /><h2>2. Including Vue into the Project</h2>I'm going to implement a web part as a Single File Component (SFC) with a help of <a href="https://github.com/vuejs/vue-class-component" target="_blank">vue-class-component</a>. This is one of two available options how to implement an SFC using Vue.js. Another one is to use <span class="code">Vue.extend</span> which is more "classic" way for Vue developers. But <span class="code">vue-class-component</span> is more similar to SPFx development itself. So, as mentioned, I'll use this one.<br />First of all, we need to install <span class="code">vue</span> module as we did in the first post: <br />
<div markdown="1">
{% highlight console %}
npm i --save vue

{% endhighlight %}
</div>
Next, we're going to create SFC and implement components inside <span class="code">.vue</span> files. To let webpack know about this type of files we need to install <a href="https://github.com/vuejs/vue-loader" target="_blank">vue-loader</a>. Also, we need to install <span class="style">vue-template-compiler</span> as it is a dependency of <span class="code">vue-loader</span>.<br />Additionally, we need <span class="code">ts-loader</span> and <span class="code">sass-loader</span> to correctly process TypeScript code and SCSS styles inside SFCs.<br />We need these modules for bundling only. That's why we'll install them with <span class="code">--save-dev</span> parameter: <br />
<div markdown="1">
{% highlight console %}
npm i --save-dev vue-loader vue-template-compiler sass-loader ts-loader

{% endhighlight %}
</div>
Now, as mentioned, we need <span class="code">vue-class-component</span> and additionally <a href="https://github.com/kaorun343/vue-property-decorator" target="_blank">vue-property-decorator</a> to mark Vue component properties: <br />
<div markdown="1">
{% highlight console %}
npm i --save vue-class-component vue-property-decorator

{% endhighlight %}
</div>
And one more thing...<br />By default, <span class="code">ts-loader</span> creates additional assets that are not recognized by SharePoint Framework in release mode. It leads to errors in <span class="code">gulp bundle --ship</span> command and inability to package the solution.<br />To avoid that we can use <span class="code">transpileOnly: true</span> option of <span class="code">ts-loader</span> (see Webpack config modifications below). But it leads to disabling of type and syntactic checking inside <span class="code">.vue</span> files.<br />To keep the checking we can use <a href="https://www.npmjs.com/package/fork-ts-checker-webpack-plugin" target="_blank">fork-ts-checker-webpack-plugin</a>: 
<div markdown="1">
{% highlight console %}

npm i --save-dev fork-ts-checker-webpack-plugin

{% endhighlight %}
</div>
And combining all the modules: <br />
<div markdown="1">
{% highlight console %}
npm i --save vue vue-class-component vue-property-decorator
npm i --save-dev vue-loader vue-property-decorator sass-loader ts-loader fork-ts-checker-webpack-plugin

{% endhighlight %}
</div>
<h2>3. Modifying Webpack config and Gulp tasks</h2>Next step is to modify <span class="code">gulpfile.js</span>. We need to change multiple things to build our Vue components correctly: <br /><ol><li>Copy Vue files to the destination folder.</li><li>apply correct loaders for <span class="code">.vue</span> files</li></ol>The first step is to copy <span class="code">.vue</span> files to the destination folder.<br />For that we can create a Gulp sub task and add it after the TypeScript compiler task: <br />
<div markdown="1">
{% highlight javascript %}
let copyVueFiles = build.subTask('copy-vue-files', function(gulp, buildOptions, done){
  return gulp.src(['src/**/*.vue'])
           .pipe(gulp.dest(buildOptions.libFolder))
});
build.rig.addPostTypescriptTask(copyVueFiles);

{% endhighlight %}
</div>
With version 1.6 of SharePoint Framework we can also create a task to initiate build when we're in debug session (<span class="code">gulp serve</span>) and <span class="code">.vue</span> file has been saved.<br />For that we can use next flow: add watch task on <span class="code">.vue</span> files. When watch fires event that the files has been saved we're copying empty <span class="code">index.ts</span> file from <span class="code">src</span> folder to itself. It will kick of the build process.<br />Thanks to <a href="https://n8d.at/blog/" target="_blank">Stefan Bauer</a> for the idea. 
<div markdown="1">
{% highlight javascript %}

// marker to check if custom watch is already registered
// used to prevent watch bubbling
let customWatchRegistered = false;

let watchVueFiles = build.subTask('watch-vue-files', function (gulp, buildOptions, done) {
    // register watch only on first run
    if (!customWatchRegistered) {

        // on change of *.vue files
        gulp.watch('./src/**/*.vue', event => {
            // copy empty index.ts onto itself to launch build procees
            gulp.src('./src/index.ts')
                .pipe(gulp.dest('./src/'));
        });

        // after watch is registered don't register again
        customWatchRegistered = true;

    }

    done();
});

build.rig.addPreBuildTask(watchVueFiles);

{% endhighlight %}
</div>
Next thing to do is to modify Webpack config to use vue-loader, ts-loader and sass-loader for <span class="code">.vue</span> files and Fork TS Checker plugin for type and syntactical checking: 
<div markdown="1">
{% highlight typescript %}
// Merge custom loader to web pack configuration
build.configureWebpack.mergeConfig({

    additionalConfiguration: (generatedConfiguration) => {

        const VueLoaderPlugin = require('vue-loader/lib/plugin');
        const ForkTsCheckerWebpackPlugin = require('fork-ts-checker-webpack-plugin');

        const vuePlugin = new VueLoaderPlugin();
        const forkTsPlugin = new ForkTsCheckerWebpackPlugin({
            vue: true,
            tslint: true,
            formatter: 'codeframe',
            checkSyntacticErrors: false
          });

        const loadersConfigs = [{
            test: /\.vue$/, // vue
            use: [{
                loader: 'vue-loader'
            }]
        }, {
            resourceQuery: /vue&type=script&lang=ts/, // typescript
            loader: 'ts-loader',
            options: {
                appendTsSuffixTo: [/\.vue$/],
                transpileOnly: true
            }
        }, {
            resourceQuery: /vue&type=style.*&lang=scss/, // scss
            use: [
                {
                    loader: require.resolve('@microsoft/loader-load-themed-styles'),
                    options: {
                        async: true
                    }
                },
                {
                    loader: 'css-loader',
                    options: {
                        modules: true,
                        localIdentName: '[local]_[sha1:hash:hex:8]'
                    }
                },
                'sass-loader']
        }, {
            resourceQuery: /vue&type=style.*&lang=sass/, // sass
            use: [
                {
                    loader: require.resolve('@microsoft/loader-load-themed-styles'),
                    options: {
                        async: true
                    }
                },
                {
                    loader: 'css-loader',
                    options: {
                        modules: true,
                        localIdentName: '[local]_[sha1:hash:hex:8]'
                    }
                },
            'sass-loader?indentedSyntax']  
        }];

        generatedConfiguration.plugins.push(vuePlugin, forkTsPlugin);
        generatedConfiguration.module.rules.push(...loadersConfigs);

        return generatedConfiguration;

    }
});
{% endhighlight %}
</div>
Few words about the code above<br />First, we're instantiating 2 plugins: <ul><li><span class="code">VuePlugin</span> to correctly kick off processing of <span class="code">.vue</span> files by <span class="code">vue-loader</span> and</li><li><span class="code">ForkTsCheckerWebpackPlugin</span> to check typings in <span class="code">.vue</span> files and use tslint rules for syntactical checking</li></ul>Next, we're adding loader configurations. <ul><li><span class="code">vue-loader</span> to process <span class="code">.vue</span> files</li><li><span class="code">ts-loader</span> to process TypeScript code from <span class="code">.vue</span> files. The interesting thing here is that we don't want to process <span class="code">.ts</span> files with <span class="code">ts-loader</span>. To avoid that we're using <span class="code">resourceQuery</span> rule to process only code from files that have <span class="code">vue&type=script&lang=ts</span> in query string parameters. These parameters are added by <span class="code">vue-loader</span> for <span class="code">&lt;script lang="ts"&gt;</span> blocks</li><li>For<span class="code">&ltstyle&gt;</span> blocks we're adding multiple loaders to work sequentially: <span class="code">@microsoft/loader-load-themed-styles</span> loader to process <a href="http://blog.aterentiev.com/2017/04/using-sharepoint-themes-colors-in-spfx.html" target="_blank">theme variables</a>;<span class="code">css-loader</span> to be able to use <a href="https://github.com/css-modules/css-modules" target="_blank">CSS Modules</a>; and <span class="code">sass-loader</span> to process scss or sass if needed.</li></ul>With these configurations in place <span class="code">.vue</span> files will be processed correctly. <h2>4. Implementing Vue Single File Component</h2>Now we're ready to implement web part's markup as a Vue SFC.<br />For that let's create <span class="code">components</span> folder, <span class="code">SimpleWebPart</span> subfolder, and <span class="code">SimpleWebPart.vue</span> file in it.<br />Now let's move the markup from the Web Part <span class="code">.ts</span> file to <span class="code">.vue</span> file.<br />The markup (HTML) should be placed inside <span class="code">template</span> element. So, after copying, the <span class="code">.vue</span> file should look like this: <br />
<div markdown="1">
{% highlight html %}
<template>
    <div class="${ styles.vueSimpleWp }">
        <div class="${ styles.container }">
            <div class="${ styles.row }">
                <div class="${ styles.column }">
                    <span class="${ styles.title }">Welcome to SharePoint!</span>
                    <p class="${ styles.subTitle }">Customize SharePoint experiences using Web Parts.</p>
                    <p class="${ styles.description }">${escape(this.properties.description)}</p>
                    <a href="https://aka.ms/spfx" class="${ styles.button }">
                        <span class="${ styles.label }">Learn more</span>
                    </a>
                </div>
            </div>
        </div>
    </div>
</template>

{% endhighlight %}
</div>
Next step in SFC creation is to add "model" - the Component class that contains properties, events handlers and business logic.<br />For that we need to add <span class="code">&lt;script lang="ts"&gt;</span> section in <span class="code">.vue</span> file and add our TypeScript code there. The code should contain a "component" class (marked with <span class="code">@Component</span> attribute and extend <span class="code">Vue</span> interface.<br />To be more close to SPFx development standarts, I will also define <span class="code">ISimpleWebPartProps</span> interface to contain component's properties declaration: 
<div markdown="1">
{% highlight typescript %}

<script lang="ts">
import { Vue, Component, Prop, Provide } from 'vue-property-decorator';

/**
 * Component's properties
 */
export interface ISimpleWebPartProps {
    description: string;
}

/**
 * Class-component
 */
@Component
export default class SimpleWebPart extends Vue implements ISimpleWebPartProps {

    /**
     * implementing ISimpleWebPartProps interface
     */
    @Prop()
    public description: string;
}
</script>

{% endhighlight %}
</div>
<b>Note:</b> you can implement TypeScript logic in separate <span class="code">.ts</span> file and reference it inside <span class="code">.vue</span> using <span class="code">src</span> attribute: 
<div markdown="1">
{% highlight html %}

<script src="./YourFile.ts">
</script>

{% endhighlight %}
</div>
After implementing the Component class we can reference <span class="code">description</span> property in our template.<br />For that we need to use mustache syntax and replace <span class="code">${escape(this.properties.description)}</span> with <span class="code">{{ "{{" }}description}}</span>: 
<div markdown="1">
{% highlight html %}

<p class="${ styles.description }">{{ "{{" }}description}}</p>

{% endhighlight %}
</div>
Now, let's copy web part styles to our component in <span class="code">&lt;style lang="scss"&gt;</span>section.<br />One change to be done in copied styles is to change <span class="code">@import</span> statement: file name should be changed from <span class="code">SPFabricCore.scss</span> to <span class="code">_SPFabricCore.scss</span>. The reason for that is that physically there is no <span class="code">SPFabricCore.scss</span> file in SP Office UI Fabric Core module. This name is process by custom SharePoint Framework Gulp task and resolved correctly to actual file <span class="code">_SPFabricCore.scss</span>. In our case we don't have access to that custom Gulp task and need to reference actual file directly.<br />The code in <span class="code">style</span> section should look like that: 
<div markdown="1">
{% highlight html %}

<style lang="scss" module>
@import "~@microsoft/sp-office-ui-fabric-core/dist/sass/_SPFabricCore.scss";

.vueSimpleWp {
  .container {
    max-width: 700px;
    margin: 0px auto;
    box-shadow: 0 2px 4px 0 rgba(0, 0, 0, 0.2), 0 25px 50px 0 rgba(0, 0, 0, 0.1);
  }

  .row {
    @include ms-Grid-row;
    @include ms-fontColor-white;
    background-color: $ms-color-themeDark;
    padding: 20px;
  }

  .column {
    @include ms-Grid-col;
    @include ms-lg10;
    @include ms-xl8;
    @include ms-xlPush2;
    @include ms-lgPush1;
  }

  .title {
    @include ms-font-xl;
    @include ms-fontColor-white;
  }

  .subTitle {
    @include ms-font-l;
    @include ms-fontColor-white;
  }

  .description {
    @include ms-font-l;
    @include ms-fontColor-white;
  }

  .button {
    // Our button
    text-decoration: none;
    height: 32px;

    // Primary Button
    min-width: 80px;
    background-color: $ms-color-themePrimary;
    border-color: $ms-color-themePrimary;
    color: $ms-color-white;

    // Basic Button
    outline: transparent;
    position: relative;
    font-family: "Segoe UI WestEuropean", "Segoe UI", -apple-system,
      BlinkMacSystemFont, Roboto, "Helvetica Neue", sans-serif;
    -webkit-font-smoothing: antialiased;
    font-size: $ms-font-size-m;
    font-weight: $ms-font-weight-regular;
    border-width: 0;
    text-align: center;
    cursor: pointer;
    display: inline-block;
    padding: 0 16px;

    .label {
      font-weight: $ms-font-weight-semibold;
      font-size: $ms-font-size-m;
      height: 32px;
      line-height: 32px;
      margin: 0 4px;
      vertical-align: top;
      display: inline-block;
    }
  }
}
</style>

{% endhighlight %}
</div>
We're also using <span class="code">module</span> attribute to use CSS Modules.<br />Now the styles can be used in the template (in markup) using pre-defined <span class="code">$style</span> variable: 
<div markdown="1">
{% highlight html %}
<template>
    <div :class="$style.vueSimpleWp">
        <div :class="$style.container">
            <div :class="$style.row">
                <div :class="$style.column">
                    <span :class="$style.title">Welcome to SharePoint!</span>
                    <p :class="$style.subTitle">Customize SharePoint experiences using Web Parts.</p>
                    <p :class="$style.description">{{ "{{" }}description}}</p>
                    <a href="https://aka.ms/spfx" :class="$style.button">
                        <span :class="$style.label">Learn more</span>
                    </a>
                </div>
            </div>
        </div>
    </div>
</template>

{% endhighlight %}
</div>
<span class="code">:class</span> syntax here is a shorthand for <span class="code">v-bind:class</span> that allows to bind property to the <span class="code">class</span> HTML attribute. <h2>5. Adding Vue SFC Component in Web Part</h2>The last step is to add created element instead default HTML in the Web Part.<br />For that, let's import <span class="code">Vue</span> object, our component and the properties: <br />
<div markdown="1">
{% highlight typescript %}
import Vue from 'vue';
import SimpleWebPart, { IVueSimpleWpWebPartProps } from './components/SimpleWebPart/SimpleWebPart.vue';

{% endhighlight %}
</div>
And now let's add the component to markup and provide <span class="code">description</span> value from the Web Part's properties. <br />
<div markdown="1">
{% highlight typescript %}
public render(): void {
  const id: string = `wp-${this.instanceId}`;
  this.domElement.innerHTML = `<div id="${id}"></div>`;

  let el = new Vue({
    el: `#${id}`,
    render: h => h(SimpleWebPartComponent, {
      props: {
        description: this.properties.description
      }
    })
  });
}

{% endhighlight %}
</div>
I'm using unique id in the <span class="code">div</span> element to inject Vue component.<br />Usually it's not a good idea to use ids for any references, but here I've tried to reference directly <span class="code">this.domElement</span> in the Vue constructor. But in that case "reactive" change of web part's properties will not work.<br />So I decided to go with id.<br />If you have any other ideas - feel free to share!<br />Now, if you try to compile the project with <span class="code">gulp</span> command, you'll receive the error <br />
<div markdown="1">
{% highlight console %}
Error - typescript - src/webparts/vueSimpleWp/VueSimpleWpWebPart.ts(16,35): error TS2307: Cannot find module './components/SimpleWebPart/SimpleWebPart.vue'.

{% endhighlight %}
</div>
That is because TypeScript compiler doesn't know how <span class="code">.vue</span> files look like when they're imported.<br />To notify TypeScript about the structure of <span class="code">.vue</span> files (and modules) we need to add <span class="code">vue-shims.d.ts</span> file in the <span class="code">src</span> folder with the next content: <br />
<div markdown="1">
{% highlight typescript %}
// src/vue-shims.d.ts

declare module "*.vue" {
    import Vue from "vue";
    export default Vue;
}

{% endhighlight %}
</div>
Now everything should work fine and you should see a standard web part markup on your page:<br /><img border="0" src="{{site.baseurl}}/assets/images/posts/2018/simple-web-part.png" /><br />The code for this example is available <a href="https://github.com/AJIXuMuK/vuejs/tree/master/simple-web-part" target="_blank">here</a><br />That's it for now!<br />Have fun and stay tuned!
