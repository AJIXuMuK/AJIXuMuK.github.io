---
layout: post
title: Handling Multiple Editions of SPFx Solution
date: '2017-08-23T15:57:00.000-07:00'
author: Alex Terentiev
tags:
- SharePoint Online
- Gulp
- O365
- SharePoint Framework
- Office 365
- SharePoint
modified_time: '2017-08-23T16:04:04.065-07:00'
blogger_id: tag:blogger.com,1999:blog-3066084330774405472.post-2919549689913343153
blogger_orig_url: http://blog.aterentiev.com/2017/08/handling-multiple-editions-of-spfx.html
---

In this post I want to explain an approach of how to handle (generate builds, update customers) multiple editions of the same SharePoint Framework solutions.<br />Use case for the approach:<br />You are an ISV and developing some product that has multiple editions, let's say, trial, lite, full. You want to have separate package file (sppkg) for each edition and also reference different CDNs based on the edition. <br />The approach I'll describe is not the only possible but it was successfully used for our own products.<br /><a name='more'></a>Thinking about the problem in details we can point several key features that should be implemented: <br /><ul><li>When we're creating a new version we should create 3 separate sppkg files</li><li>Each sppkg file should contain manifest that references different CDN endpoints</li><li>It should be easy to upgrade customer from trial to lite and then to full; or directly from trial to full</li><li>You should know current edition in the code to execute the logic based on the edition's restrictions</li></ul>Let's look at each point separately and describe what should be done to implement needed behavior. <br /><h4>When we're creating a new version we should create 3 separate sppkg files</h4>The path to the sppkg file is configured in <span class="code">config/package-solution.json</span> file in <span class="code">paths.zippedPackage</span> property. By default, it contains <span class="code">solution/&lt;name-of-the-solution&gt;.sppkg</span> which means that the file will be created in <span class="code">sharepoint/solution</span> folder.<br />The first idea was to have different filename for each edition, like <span class="code">product-trial.sppkg</span>, <span class="code">product-lite.sppkg</span> and <span class="code">product-full.sppkg</span>. But it will not work as you can't deploy both packages to App Catalog. You'll receive the exception that two different packages can't contain the same solution.<br />So the other approach is to create subfolders for each version, like <span class="code">solution/trial/product.sppkg</span>, <span class="code">solution/lite/product.sppkg</span> and <span class="code">solution/full/product.sppkg</span><br /><h4>Each sppkg file should contain manifest that references different CDN endpoints</h4>CDN endpoint is <span class="code">config/write-manifests.json</span> file in <span class="code">cdnBasePath</span> property. And also in <span class="code">config/deploy-azure-storage.json</span> in <span class="code">container</span> property - this should be also taken into consideration if you're going to deploy the assets to the Azure.<br />So, we can change these properties to reference different CDN endpoints. <br /><h4>It should be easy to upgrade customer from trial to lite and then to full; or directly from trial to full</h4>Currently app updates are done based on the value of <span class="code">solution.version</span> property in <span class="code">config/package-solution.json</span> file.<br />If you look at Apps for SharePoint list in the App Catalog you'll see that each deployed package (or app) has App Version value. If you install a newer version of the package SharePoint will mark the app as needed for update. And later a user can get newer version of the app on specific site (or the app will be updated everywhere automatically if <a href="https://dev.office.com/sharepoint/docs/spfx/tenant-scoped-deployment">tenant-scoped deployment</a> is enabled.<br />The <span class="code">solution.version</span> has 4-digit signature (as everything in Microsoft Universe): <span class="code">major.minor.build.revision</span>. So, we can use revision to address each of our editions: <br /><ul><li>0 for trial</li><li>1 for lite</li><li>2 for full</li></ul>It will allow us to update any customer from, let's say, trial to full by providing a solution package that has the same major, minor and build digits but 2 instead on 0 in revision digit. As 2 &gt; 0 it will mark the app as needed for update. <br /><h4>You should know current edition in the code to execute the logic based on the edition's restrictions</h4>The idea here is to have some configuration file that is referenced in the code and the content if which differs for each edition. In our product we created a file <span class="code">custom-config.json</span> in web part folder that contains JSON with current edition: <br />
<div markdown="1">
{% highlight javascript %}
{
  "edition": "full"
}

{% endhighlight %}
</div>
And then the file is referenced in web part's code: <br />
<div markdown="1">
{% highlight typescript %}
var customConfig: any = require('./custom-config.json');

//...

switch (customConfig.edition) {
  // implement needed cases
}

{% endhighlight %}
</div>
<h4>Combining all in one</h4>We addressed all the points but still the process is totally manual. It would be great to automate needed changes. And it can be done with custom Gulp tasks.<br />First of all, let's create a json file to contain settings for each edition. Needed settings are: <br /><ul><li>edition name</li><li>azure container</li><li>CDN path</li><li>revision number (0, 1 or 2 in our case)</li><li>package path</li></ul>Let's name the file <span class="code">build-config.json</span> and add it to config folder. The content of the file should be like that: <br />
<div markdown="1">
{% highlight javascript %}
{
  "editions": [{
    "edition": "trial",
    "azureContainer": "js-solution-editions-trial",
    "cdnPath": "<!-- PATH TO CDN TRIAL -->",
    "revision": "0",
    "pkgPath": "solution/trail/js-solution-editions.sppkg"
  }, {
    "edition": "lite",
    "azureContainer": "js-solution-editions-lite",
    "cdnPath": "<!-- PATH TO CDN LITE -->",
    "revision": "1",
    "pkgPath": "solution/lite/js-solution-editions.sppkg"
  }, {
    "edition": "full",
    "azureContainer": "js-solution-editions-full",
    "cdnPath": "<!-- PATH TO CDN TRIAL -->",
    "revision": "2",
    "pkgPath": "solution/full/js-solution-editions.sppkg"
  }]
}

{% endhighlight %}
</div>
Now we can create a Gulp task that will take edition as a parameter and will modify configuration files base on edition settings. The task should be added to <span class="code">gulpfile.js</span> before global initialization of the build (it means before line <span class="code">build.initialize(gulp)</span>). You can read more about custom Gulp tasks in SPFx <a href="https://dev.office.com/sharepoint/docs/spfx/toolchain/integrate-gulp-tasks-in-build-pipeline" target="_blank">here</a>.<br />The steps in the task will address all the points addressed above: <br /><ul><li>Modify <span class="code">config/package-solution.json</span> to change revision number in <span class="code">solution.version</span> property and also change <span class="code">paths.zippedPackage</span> property. It worth saying that right now <span class="code">gulp package-solution</span> fails if <span class="code">zippedPackage</span> contains path to the folder that doesn't exist. That's why we'll need to additionally 'ensure' path to the sppkg. In our case we'll check if there is trail|lite|full subfolder in <span class="code">sharepoint/solution</span> and create it if needed.</li><li>Modify <span class="code">config/write-manifests.json</span> to change CDN endpoint url</li><li>Modify <span class="code">config/deploy-azure-storage.json</span> to change Azure Storage container, and</li><li>Modify <span class="code">custom-config.json</span> file in web part's source code folder to set current edition</li></ul>Full code of <span class="code">gulpfile.js</span> is listed below: <br />
<div markdown="1">
{% highlight typescript %}
'use strict';

const gulp = require('gulp');
const build = require('@microsoft/sp-build-web');

const logging = require('@microsoft/gulp-core-build');
const fs = require('fs');

// path to editions config file
const buildConfigFilePath = './config/build-config.json';
// path to deploy-azure-storage.json
const azureConfigFilePath = './config/deploy-azure-storage.json';
// path to package-solution.json
const solutionConfigFilePath = './config/package-solution.json';
// path to write-manifests.json
const manifestFilePath = './config/write-manifests.json';
// path to custom-config.json that contains edition name to use in code
const customConfigFilePath = './src/webparts/helloWorld/custom-config.json';

// adding custom task. Can be executed with gulp change-build-edition --edition lite
build.task('change-build-edition', {
  execute: (config) => {
    return new Promise((resolve, reject) => {
      try {
        // getting edition parameter
        const edition = config.args['edition'] || 'full';

        // getting package-solution.json content
        const solutionJSON = JSON.parse(fs.readFileSync(solutionConfigFilePath));
        // getting deploy-azure-storage.json content
        const azureJSON = JSON.parse(fs.readFileSync(azureConfigFilePath));
        // getting write-manifests.json content
        const manifestJSON = JSON.parse(fs.readFileSync(manifestFilePath));
        // getting custom-config.json content
        const customConfigJSON = JSON.parse(fs.readFileSync(customConfigFilePath));
        // getting editions configurations
        const buildJSON = JSON.parse(fs.readFileSync(buildConfigFilePath));

        // getting edition settings by edition name
        const editionInfo = getEditionInfo(buildJSON, edition);

        if (!editionInfo) {
         resolve();
         return;
        }

        logging.log(`Configuring settings for edition: ${edition}`);

        //
        // updating custom-config.json file
        //
        customConfigJSON.edition = edition;
        logging.log('Updating custom config for the web part...');
        fs.writeFileSync(customConfigFilePath, JSON.stringify(customConfigJSON));

        //
        // updating package-solution.json
        //
        const revNumberStartIndex = solutionJSON.solution.version.lastIndexOf('.');
        // new version
        solutionJSON.solution.version = solutionJSON.solution.version.substring(0, revNumberStartIndex + 1) + editionInfo.revision;
        logging.log(`Checking if sppkg directory '${editionInfo.pkgPath}' exists and creating if not...`);
        // creating subfolder if doesn't exist
        ensurePath(editionInfo.pkgPath);
        // updating zippedPackage path
        solutionJSON.paths.zippedPackage = editionInfo.pkgPath;
        logging.log('Updating package-solution.json...');
        fs.writeFileSync(solutionConfigFilePath, JSON.stringify(solutionJSON));

        //
        // updating deploy-azure-storage.json
        //
        azureJSON.container = editionInfo.azureContainer;
        logging.log('Updating deploy-azure-storage.json...');
        fs.writeFileSync(azureConfigFilePath, JSON.stringify(azureJSON));

        //
        // updating write-manifests.json
        //
        manifestJSON.cdnBasePath = editionInfo.cdnPath;
        logging.log('Updating write-manifests.json...');
        fs.writeFileSync(manifestFilePath, JSON.stringify(manifestJSON));

        resolve();
      }
      catch (ex) {
        logging.log(ex);
        reject();
      }
    });
  }
});

/**
 * Gets edition settings by name
 * @param {any} buildJSON editions settings
 * @param {string} edition edition name
 */
function getEditionInfo(buildJSON, edition) {
  edition = edition || 'full';
  let result = null;

  if (buildJSON &amp;&amp; buildJSON.editions &amp;&amp; buildJSON.editions.length) {
    for (let i = 0, len = buildJSON.editions.length; i < len; i++) {
      const ver = buildJSON.editions[i];
      if (ver.edition === edition) {
        result = ver;
        break;
      }
    }
  }

  if (!result) {
    result = {
      'edition': 'full',
      'azureContainer': 'js-solution-editions-full',
      'cdnPath': '<!-- PATH TO CDN FULL -->',
      'revision': '2',
      'pkgPath': 'solution/full/js-solution-editions.sppkg'
    };
  }

  return result;
}

/**
 * Ensures that the subfolders from the path exist
 * @param {string} path relative path to sppkg file (relative to ./sharepoint folder)
 */
function ensurePath(path) {
  if (!path) {
    return;
  }

  let pathArray = path.split('/');
  if (!pathArray.length) {
      return;
  }


  //
  // removing filename from the path
  //
  if (pathArray[pathArray.length - 1].indexOf('.') !== -1) {
      pathArray.pop();
  }

  //
  // adding sharepoint as a root folder
  //
  if (pathArray[0] !== 'sharepoint') {
      pathArray.unshift('sharepoint');
  }

  //
  // creating all subfolders if needed
  //
  let currPath = '.';

  for (let i = 0, length = pathArray.length; i < length; i++) {
    const pathPart = pathArray[i];
    currPath += `/${pathPart}`;

    if (!fs.existsSync(currPath)) {
      fs.mkdir(currPath);
    }
  }
}

build.initialize(gulp);

{% endhighlight %}
</div>
Now you can run this task as <br />
<div markdown="1">
{% highlight console %}
gulp change-build-edition --edition lite

{% endhighlight %}
</div>
And then run out of the box <br />
<div markdown="1">
{% highlight console %}
gulp bundle --ship
gulp package-solution --ship
gulp deploy-azure-storage

{% endhighlight %}
</div>
to package specific version.<br />After you run these for tasks for each edition you'll have 3 sppkg files in separate subfolder and also referencing CDN endpoints with different <span class="code">custom-config.json</span> content. And consequently edition-based behavior.<br />Now you can deploy trial or lite edition to App Catalog and then easily upgrade to full.<br />Hope this post will help developing SPFx 3d party products.<br />Have fun!