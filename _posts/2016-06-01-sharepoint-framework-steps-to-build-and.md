---
layout: post
title: SharePoint Framework - Steps to build and run your first project
date: '2016-06-01T22:53:00.000-07:00'
author: Alex Terentiev
tags:
- SharePoint Online
- Grunt
- npm
- Gulp.js
- Grunt.js
- Gulp
- Node.js
- SharePoint Framework
- Office 365
- Node
- SharePoint
- yo
- Yeoman
- generator-sharepoint
- Future of SharePoint
modified_time: '2016-08-18T15:57:10.498-07:00'
thumbnail: https://1.bp.blogspot.com/-g2o_ZXGDup8/V0-0rPJqCQI/AAAAAAAAAX8/Yl3N2u-va_06Pvjp6W-aX98njWatvdh4QCLcB/s72-c/Screen%2BShot%2B2016-06-01%2Bat%2B9.22.31%2BPM.png
blogger_id: tag:blogger.com,1999:blog-3066084330774405472.post-525224568421759804
blogger_orig_url: http://blog.aterentiev.com/2016/06/sharepoint-framework-steps-to-build-and.html
---

<b>UPDATE 8/18/2016 - </b>Microsoft&nbsp;<a href="https://dev.office.com/blogs/sharepoint-framework-developer-preview-release" target="_blank">announced</a>&nbsp;SharePoint Framework Developer Release.<br /><br /><br /><b>SPOILER: </b>at the end of this post you'll be able to run a web app that is provided by Yeoman generator for SharePoint. But as of now (6/2/16) it's a simple web app based on Grunt for build and Mocha/PhantomJS to run. So you won't see Client Web Part deployed to SharePoint Workbench (I think that Microsoft should update the generator soon). Still this post may be useful for people who didn't work with Node.js and front-end stacks.<br /><br />Here I want to provide all the steps you need to run build and run your first SharePoint Framework project on <b>Windows </b>machine (it's really easier on Mac).<br /><div>You can find some steps <a href="https://docs.com/wictor/5207/sharepoint-framework-your-first-glance-into-the?c=sS1taT" target="_blank">here</a>&nbsp;but after following them you won't be able to run the project (or even compile/build).</div><div>So let's start.</div><div><a name='more'></a><ol><li>Install node.js from&nbsp;<a href="https://nodejs.org/en/download/" target="_blank">here</a></li><li>Install <a href="http://www.ruby-lang.org/" target="_blank">Ruby</a> using <a href="http://rubyinstaller.org/downloads/" target="_blank">RubyInstaller</a>. Select "Add Ruby executables to your PATH"<div class="separator" style="clear: both; text-align: center;"><img src="{{site.baseurl}}/assets/images/posts/2016/2016-06-01.png" /></div></li><li>Install <a href="https://code.visualstudio.com/" target="_blank">Visual Studio Code</a>&nbsp;or any other IDE</li><li>Restart your computer (VM)</li><li>Update Ruby (here and further provided the commands for cmd):<pre style="background: black; color: white;">gem update --system</pre></li><li>Install <a href="http://compass-style.org/" target="_blank">Compass</a>:<pre style="background: black; color: white;">gem install compass</pre></li><li>Install <a href="http://yeoman.io/" target="_blank">Yeoman</a>:<pre style="background: black; color: white;">npm install -global yo</pre></li><li>Install <a href="http://gulpjs.com/">Gulp</a>(everywhere is said that SharePoint Framework will use Gulp to build projects but now it's using Grunt so see next step):<pre style="background: black; color: white;">npm install -global gulp</pre></li><li>Install <a href="http://gruntjs.com/">Grunt</a>:<pre style="background: black; color: white;">npm install -global grunt</pre></li><li>Install SharePoint Generator:<pre style="background: black; color: white;">npm install -global generator-sharepoint</pre></li><li>Install <a href="https://github.com/yeoman/generator-mocha" target="_blank">generator</a> for <a href="https://mochajs.org/" target="_blank">Mocha</a> app:<pre style="background: black; color: white;">npm install -global generator-mocha</pre></li><li>Install <a href="http://bower.io/" target="_blank">Bower</a>:<pre style="background: black; color: white;">npm install -global bower</pre></li><li>Create a folder where you want to work with your project</li><li>Scaffold your SharePoint project:<pre style="background: black; color: white;">yo sharepoint</pre>(I selected Yes to Sass and RequireJS and BDD for type of project) </li><li>Open your folder in Visual Studio Code</li><li>Uninstall <i>grunt-requirejs</i> module and install <i>grunt-contrib-requirejs</i> module:<pre style="background: black; color: white;">npm uninstall grunt-requirejs --save-dev<br />npm install grunt-contrib-requirejs --save-dev</pre></li><li>Reinstall <i>grunt-mocha</i> module:<pre style="background: black; color: white;">npm uninstall grunt-mocha --save-dev<br />npm install grunt-mocha --save-dev</pre></li><li>Finally install all the modules with:<pre style="background: black; color: white;">npm install</pre></li><li>Run <pre style="background: black; color: white;">gulp server</pre></li></ol>I thought that after all these actions I will see something similar to announced SharePoint Workbench. But for now it looks like some stub (Yeoman template is a stub) that shows simple html page that has nothing of SharePoint. <br />I will try to update this post if something new appears.</div>