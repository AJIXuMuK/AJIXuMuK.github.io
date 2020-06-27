---
layout: post
title: 'SharePoint Framework Extensions: Application Customizer'
date: '2017-08-21T22:03:00.001-07:00'
author: Alex Terentiev
tags:
- SharePoint Online
- npm
- Gulp
- Node.js
- SharePoint Framework
- Office 365
- SharePoint Framework Extensions
- SharePoint
- SPFx
- Application Customizer
- Yeoman
- generator-sharepoint
- O365
modified_time: '2017-10-19T16:41:17.779-07:00'
thumbnail: https://3.bp.blogspot.com/-9IV7QWMfOh8/WZYX_v649II/AAAAAAAAAio/x79Ml5pC8B0FDjPoQgtrS7crxD6tbYmmQCLcBGAs/s72-c/AppCustomizer.png
blogger_id: tag:blogger.com,1999:blog-3066084330774405472.post-8661705553480101867
blogger_orig_url: http://blog.aterentiev.com/2017/08/sharepoint-framework-extensions.html
---

 In this post I want to show you step-by-step implementation of SharePoint Framework Application Customizer.<br /><b>UPDATE:</b> on 9/25/2017 SPFx Extensions were GA-ed. The post is updated to contain the latest version of API.<br /><b>UPDATE:</b> on 8/29/2017 Release Candidate was announced. The post is updated to contain newer version of API.<br />Use Case for the sample:<br />Your team is working on some project and you're using SharePoint Tasks list to assign and monitor tasks in the project. You want to notify a member of your team that he or she has overdue tasks. You want this notification to be available on any page on your site and also to provide link to the Tasks list view filtered by current member and by due date.<br />The result may look like that:<br /><div class="separator" style="clear: both; text-align: center;"><a href="https://3.bp.blogspot.com/-9IV7QWMfOh8/WZYX_v649II/AAAAAAAAAio/x79Ml5pC8B0FDjPoQgtrS7crxD6tbYmmQCLcBGAs/s1600/AppCustomizer.png" imageanchor="1" style="margin-left: 1em; margin-right: 1em;"><img border="0" data-original-height="389" data-original-width="1600" height="153" src="https://3.bp.blogspot.com/-9IV7QWMfOh8/WZYX_v649II/AAAAAAAAAio/x79Ml5pC8B0FDjPoQgtrS7crxD6tbYmmQCLcBGAs/s640/AppCustomizer.png" width="640" /></a></div><br />Key features of the sample:<br /><ul><li>SPFx Application Customizer implementation</li><li>Usage of&nbsp;<a href="https://github.com/SharePoint/PnP-JS-Core">SharePoint PnP JavaScript Core Library</a></li></ul>For those who doesn't want to read the post you can get it from&nbsp;<a href="https://github.com/SharePoint/sp-dev-fx-extensions/tree/master/samples/react-application-duetasks" target="_blank">here</a>. <br /><br /><a name='more'></a><h2>Prerequsites</h2>Before following the steps in this article, be sure to proceed next 3 steps: <br /><ul><li>Get Office Developer Tenant (you can get one for free by subscribing to&nbsp;<a href="http://dev.office.com/devprogram" target="_blank">Office 365 Developer Program</a>. It's an optional step as Extensions reached General Availability but it's good to have the tenant to be able to get preview versions in future. </li><li><a href="https://dev.office.com/sharepoint/docs/spfx/set-up-your-development-environment" target="_blank">Set up your development environment</a></li><li>Create Tasks list on your tenant's site</li></ul><h2>General Information</h2><h3>Customizer Purpose</h3><ul><li>Inject custom JavaScript on the page</li><li>Insert custom HTML to well-known locations on the page - Placeholders</li></ul><h3>Entry Points</h3><ul><li><span class="code">onInit()</span> – runs code before page DOM is ready. Perform needed setup here </li><li><span class="code">this.context.application._layoutChangedEvent</span> – event that gets raised every time the layout changes in a page. It's marked as internal but still available for usage </li><li><span class="code">this.context.application.navigatedEvent</span> – event that gets raised every time there is a page navigation </li><li><span class="code">this.context.placeholderProvider.changedEvent</span> – event that gets raised when the list of currently available placeholders is changed </li></ul><h3>Helpful APIs</h3><ul><li><span class="code">this.properties</span> – properties that are passed to the customizer</li><li><span class="code">this.context.pageContext</span> – standard definitions for common SharePoint objects (site, web, user, list, etc.)</li><li><span class="code">this.context.httpClient, this.context.spHttpClient, this.context.graphHttpClient</span> – helpers to send http requests, http requests with SharePoint context and Microsoft Office Graph requests</li><li><span class="code">this.context.placeholderProvider</span> – helper object to work with placeholders.</li><li><span class="code">this.context.application</span> – helper object to work with "application". For now it can be used to attach to application's events.</li></ul><h3>Working with Placeholders</h3><ul><li>Currently available placeholders (as of Oct 2, 2017): Top, Bottom</li><li>Import PlaceholderContent and PlaceholderName definition: 
<div markdown="1">
{% highlight typescript %}
import { PlaceholderContent, PlaceholderName } from '@microsoft/sp-application-base';

{% endhighlight %}
</div>
</li><li>Get placeholder instance: 
<div markdown="1">
{% highlight typescript %}
const placeholder = this.context.placeholderProvider.tryCreateContent(PlaceholderName.Top);

{% endhighlight %}
</div>
</li><li>Insert markup: 
<div markdown="1">
{% highlight typescript %}
placeholder.domElement.innerHTML = `<div>Hello World</div>`;

{% endhighlight %}
</div>
</li></ul><h2>Scaffolding the Project</h2>First step in any SPFx project is to "scaffold" it using Yeoman SharePoint Generator. "Scaffold" in more classic terms means generate or create from a template.<br />Here are the steps to create a project.<br /><ol><li>Open Command Prompt (Terminal in MacOS) or any Console tool</li><li>Create a new directory for the project in the location you want 
<div markdown="1">
{% highlight console %}
mkdir overdue-tasks

{% endhighlight %}
</div>
</li><li>Go to the created directory 
<div markdown="1">
{% highlight console %}
cd overdue-tasks

{% endhighlight %}
</div>
</li><li>Start scaffolding new project with Yeoman SharePoint Generator 
<div markdown="1">
{% highlight console %}
yo @microsoft/sharepoint

{% endhighlight %}
</div>
</li><li>When prompted <ul><li>Accept default solution name (overdue-tasks)</li><li>For baseline packages select SharePoint Online only (latest) as SPFx Extensions are not available for SharePoint 2016</li><li>Select Use the current folder</li><li>You can select either y or N for availability of your extension across tenant. It depends on how you're going to install it. If you use Feature Framework to add application customizer then this option will not work as it doesn't work with Feature Framework</li><li>Select Extension as a client-side component</li><li>Select Application Customizer as an extension type</li><li>Type OverdueTasks as a name</li><li>Accept default description</li></ul></li><li>Wait until Yeoman is installing all needed dependencies</li></ol><a href="https://4.bp.blogspot.com/-aDuN9xeZgrw/WdJ8p3QjcOI/AAAAAAAAAkk/fma_FqaayB8WsaFzKeyBr__WS9pfVe0EgCLcBGAs/s1600/Screen%2BShot%2B2017-10-02%2Bat%2B10.47.54%2BAM.png" imageanchor="1" style="margin-left: 1em; margin-right: 1em;"><img border="0" data-original-height="605" data-original-width="1600" height="242" src="https://4.bp.blogspot.com/-aDuN9xeZgrw/WdJ8p3QjcOI/AAAAAAAAAkk/fma_FqaayB8WsaFzKeyBr__WS9pfVe0EgCLcBGAs/s640/Screen%2BShot%2B2017-10-02%2Bat%2B10.47.54%2BAM.png" width="640" /></a>Congratulations! The project has been scaffolded! Now you can open the project in favorite IDE. <br /><b>Note: </b>this is a TypeScript project so select the IDE that supports TypeScript language. I'm using&nbsp;<a href="https://code.visualstudio.com/" target="_blank">Visual Studio Code</a>.<br />The project structure should look like the one on the picture below<br /><a href="https://4.bp.blogspot.com/-1nZp6-Nr63s/WZYhlt6U9iI/AAAAAAAAAjA/YEmYHFq3jv0ngE1ca7F15DPyv9HdEqmSwCLcBGAs/s1600/Screen%2BShot%2B2017-08-17%2Bat%2B4.06.40%2BPM.png" imageanchor="1"><img border="0" data-original-height="732" data-original-width="644" height="400" src="https://4.bp.blogspot.com/-1nZp6-Nr63s/WZYhlt6U9iI/AAAAAAAAAjA/YEmYHFq3jv0ngE1ca7F15DPyv9HdEqmSwCLcBGAs/s400/Screen%2BShot%2B2017-08-17%2Bat%2B4.06.40%2BPM.png" width="351" /></a><br /><h2>Debugging the Extension using gulp serve and query string</h2>I decided to put this section at the beginning so you could run the code each time you want to check if everything works fine.<br />SharePoint Framework extensions cannot currently be tested using the local workbench, so you'll need to test and develop them directly against a live SharePoint Online site. You do not however need to deploy your customization to the app catalog to do this, which keeps the debugging experience simple and efficient. First, compile your code and host the compiled files from the local machine by running this command: <br />
<div markdown="1">
{% highlight console %}
gulp serve --nobrowser

{% endhighlight %}
</div>
<span class="code">--nobrowser</span> parameter tells to start the server but do not open workbench page in the browser. <br />If the code compiled without errors the server will be running on http://localhost:4321.<br />To test the extension<br /><ol><li>Go to SharePoint Dev tenant</li><li>Open any list/document library/modern page</li><li>Append next query string parameters to the list's URL: 
<div markdown="1">
{% highlight javascript %}
?loadSPFX=true&amp;debugManifestsFile=https://localhost:4321/temp/manifests.js&amp;customActions={"7ce714f7-704a-48a5-9a4a-9683c9f668f4":{"location":"ClientSideExtension.ApplicationCustomizer","properties":{"testMessage":"Tasks"}}}

{% endhighlight %}
</div>
Where: <ul><li><b>loadSPFX=true</b> ensures that the SharePoint Framework is loaded on the page. For performance reasons, the framework is not normally loaded unless at least one extension is registered. Since no components are registered yet, we must explicitly load the framework.</li><li><b>debugManifestsFile=&lt;URL&gt;</b> specifies that we want to load SPFx components that are being locally served. Normally the loader only looks for components in the App Catalog (for your deployed solution) and the SharePoint manifest server (for the system libraries).</li><li><b>customActions</b> simulates Custom Action. The Key here specifies the GUID of the extension that should be loaded - ID value is located in <span class="code">manifest.json</span> file of the extension. Location parameter should be <span class="code">ClientSideComponent.ApplicationCustomizer</span> for this type of customizer. The properties parameter is an optional text string containing a JSON object that will be deserialized into <span class="code">this.properties</span> for your extension (we will modify it later for the purposes of the sample).</li></ul>The full URL should be similar to following (depending on tenant URL and List URL): 
<div markdown="1">
{% highlight javascript %}
https://terentiev.sharepoint.com/Lists/Slider%20Test/AllItems.aspx?loadSPFX=true&amp;debugManifestsFile=https://localhost:4321/temp/manifests.js&amp;customActions={"7ce714f7-704a-48a5-9a4a-9683c9f668f4":{"location":"ClientSideExtension.ApplicationCustomizer","properties":{"testMessage":"Tasks"}}}

{% endhighlight %}
</div>
</li><li>Accept loading of Debug Manifests <br /><a href="https://2.bp.blogspot.com/-pz-7etinYJ0/WVwlqfDQDpI/AAAAAAAAAg0/mw96xs0nwKcVSQkmK2ZAnU_vCweqzkDwwCLcBGAs/s1600/allow-debug-scripts.png" imageanchor="1"><img border="0" data-original-height="563" data-original-width="1036" height="217" src="https://2.bp.blogspot.com/-pz-7etinYJ0/WVwlqfDQDpI/AAAAAAAAAg0/mw96xs0nwKcVSQkmK2ZAnU_vCweqzkDwwCLcBGAs/s400/allow-debug-scripts.png" width="400" /></a></li><li>You should see a popup<br /><a href="https://3.bp.blogspot.com/-TFd1G6iJYIE/WZYs0gSwMfI/AAAAAAAAAjQ/4Ahmv5caKYo2qShflcJtr1BtRKy9q-EcQCLcBGAs/s1600/Screen%2BShot%2B2017-08-17%2Bat%2B4.53.48%2BPM.png" imageanchor="1"><img border="0" data-original-height="328" data-original-width="806" height="162" src="https://3.bp.blogspot.com/-TFd1G6iJYIE/WZYs0gSwMfI/AAAAAAAAAjQ/4Ahmv5caKYo2qShflcJtr1BtRKy9q-EcQCLcBGAs/s400/Screen%2BShot%2B2017-08-17%2Bat%2B4.53.48%2BPM.png" width="400" /></a></li></ol><h2>Adding Additional Libraries</h2>Next step is to add external libraries (modules) that will be used in the project.<br />In this sample we'll need to request data from SharePoint (overdue tasks and also a URL to redirect user to the Tasks list).<br />It may be done with REST API using <span class="code">this.context.spHttpClient</span> helper. But I prefer to use SP PnP JS library which provides great syntax for generating REST requests.<br />PnP JS library can be added using npm package manager:<br /><ol><li>In Command Prompt (Terminal) go to project directory.</li><li>Install the PnP package 
<div markdown="1">
{% highlight console %}
npm install sp-pnp-js --save

{% endhighlight %}
</div>
</li></ol>Now the module can be imported to any file in the project<br />
<div markdown="1">
{% highlight typescript %}
import pnp from 'sp-pnp-js';

{% endhighlight %}
</div>
<h2>Getting Data from SharePoint</h2>As mentioned earlier we need to get overdue tasks and Tasks list url from SharePoint. And to get that data we need to know Tasks list id or Guid. It can be hardcoded but it's better to use <span class="code">this.properties</span> object for that. Let's modify <span class="code">IOverdueTasksApplicationCustomizerProperties</span> interface which describes what members will be available in <span class="code">this.properties</span>: <br />
<div markdown="1">
{% highlight typescript %}
export interface IOverdueTasksApplicationCustomizerProperties {
  tasksListTitle: string;
}

{% endhighlight %}
</div>
Also, we need to add fields to the Customizer class to store requested url and tasks: <br />
<div markdown="1">
{% highlight typescript %}
private _viewUrl: string;
private _overdueTasks: any;

{% endhighlight %}
</div>
Best place to make SharePoint requests is <span class="code">onInit</span> method. It returns <span class="code">Promise</span> object which means that the SharePoint Framework will wait until the Promise is resolve and will call <span class="code">onRender</span> only after that. So you can make all your requests and resolve the promise when you get all the responses.<br />So, all the code inside <span class="code">onInit</span> should be wrapped with the Promise constructor: <br />
<div markdown="1">
{% highlight typescript %}
public onInit(): Promise<void> {
    return new Promise((resolve) => {
      // your code goes here
    });
  }
</void>
{% endhighlight %}
</div>
As I'm going to use SP PnP JS library to make requests, I need to initialize it first with SPFx Context to provide correct SharePoint context: <br />
<div markdown="1">
{% highlight typescript %}
pnp.setup({
  spfxContext: this.context
});

{% endhighlight %}
</div>
One more great feature to use from PnP is batching: using batch object we can combine multiple requests and send it as single one. So, let's create a batch: <br />
<div markdown="1">
{% highlight typescript %}
const batch = pnp.sp.createBatch();

{% endhighlight %}
</div>
The request for view url is shown below: <br />
<div markdown="1">
{% highlight typescript %}
pnp.sp.web.lists.getByTitle(this.properties.tasksListTitle).views.getByTitle('Late Tasks').inBatch(batch).get().then((view: any) => {
  // our URL contains Late Tasks View URL and filter by current user
  this._viewUrl = `${view.ServerRelativeUrl}?FilterField1=AssignedTo&amp;FilterValue1=${escape(this.context.pageContext.user.displayName)}`;
});

{% endhighlight %}
</div>
If look at the code it is similar to constructing REST API endpoint <span class="code">/web/lists/getByTitle('&lt;title&gt;&gt;')/views/getByTitle('&lt;view_title&gt;')</span>Then the request is added to the batch.<br />And when the response is received we're constructing the Url using Late Tasks view Url and filter by current user.<br />Why Late Tasks view? Because it has filter to show overdue tasks only.<br />Additionally you may notice usage of escape function when creating filter parameters in Url Query Strings. This function is imported from <span class="code">sp-lodash-subset</span> module: <br />
<div markdown="1">
{% highlight typescript %}
import { escape } from '@microsoft/sp-lodash-subset';

{% endhighlight %}
</div>
The request from overdue tasks: <br />
<div markdown="1">
{% highlight typescript %}
let today: Date = new Date();
today.setHours(0, 0, 0, 0);

pnp.sp.web.lists.getByTitle(this.properties.tasksListTitle)
  .items.expand('AssignedTo/Id').select('Title, AssignedTo, AssignedTo/Id, DueDate')
  .filter(`AssignedTo/Id eq ${this.context.pageContext.legacyPageContext.userId} and DueDate lt datetime'${today.toISOString()}'`)
  .inBatch(batch)
  .get().then((items: any) =&amp;gt {
    this._dueTasks = items;
});

{% endhighlight %}
</div>
Similarly to view Url request we're constructing REST endpoint to get items from the list. Additionally, we're "expanding" <span class="code">AssignedTo</span> lookup field to get user id and then filtering the items by Due Date and expanded Assigned To User Id.<br />The request is also added to the batch.<br />The last command in onInit would be to execute batch. When the batch is executed we can finally call the method to render our content and resolve the promise: <br />
<div markdown="1">
{% highlight typescript %}
batch.execute().then(() => { 
  this._renderPlaceholder();
  resolve(); 
});

{% endhighlight %}
</div>
Full code of <span class="code">onInit</span> in listed below: <br />
<div markdown="1">
{% highlight typescript %}
public onInit(): Promise<void> {
  return new Promise((resolve) => {
    pnp.setup({
      spfxContext: this.context
    });

    const batch = pnp.sp.createBatch();

    pnp.sp.web.lists.getByTitle(this.properties.tasksListTitle).views.getByTitle('Late Tasks').inBatch(batch).get().then((view: any) => {
      this._viewUrl = `${view.ServerRelativeUrl}?FilterField1=AssignedTo&amp;FilterValue1=${escape(this.context.pageContext.user.displayName)}`;
    });

    let today: Date = new Date();
    today.setHours(0, 0, 0, 0);

    pnp.sp.web.lists.getByTitle(this.properties.tasksListTitle)
      .items.expand('AssignedTo/Id').select('Title, AssignedTo, AssignedTo/Id, DueDate')
      .filter(`AssignedTo/Id eq ${this.context.pageContext.legacyPageContext.userId} and DueDate lt datetime'${today.toISOString()}'`)
      .inBatch(batch)
      .get().then((items: any) => {
        this._overdueTasks = items;
    });

    batch.execute().then(() => { 
      this._renderPlaceholder();
      resolve(); 
    });
  });
}
</void>
{% endhighlight %}
</div>
Now we have all the data to display a warning message. And rendering part is done in custom <span class="code">_renderPlaceholder</span> method.<br />First thing to do here is to check if there are overdue tasks as there is no need to display message if there are no overdue tasks: <br />
<div markdown="1">
{% highlight typescript %}
if (!this._overdueTasks || !this._overdueTasks.length) {
  return;
}

{% endhighlight %}
</div>
If there are overdue tasks we need to get Top placeholder content and insert HTML markup to it. <br />First, let's import <span class="code">PlaceholderContent</span> and <span class="code">PlaceholderName</span> definitions from <span class="code">sp-application-base</span>: <br />
<div markdown="1">
{% highlight typescript %}
import {
  BaseApplicationCustomizer,
  PlaceholderContent,
  PlaceholderName
} from '@microsoft/sp-application-base';

{% endhighlight %}
</div>
Then, let's add a field to store Placeholder instance: <br />
<div markdown="1">
{% highlight typescript %}
private _topPlaceholder: PlaceholderContent;

{% endhighlight %}
</div>
Now let's get the instance in <span class="code">_renderPlaceholder</span>: <br />
<div markdown="1">
{% highlight typescript %}
if (!this._topPlaceholder) {
  this._topPlaceholder = this.context.placeholderProvider.tryCreateContent(PlaceholderName.Top, {
    onDispose: () => {}
  });
}

{% endhighlight %}
</div>
When getting the <span class="code">PlaceholderContent</span> instance you can provide <span class="code">onDispose</span> handler. It is done to be able to release allocated resources when your control is removed from the page.<br />In this sample we don't have any additional resources so the handler has no code. But it can be used, for example, to dispose React component.<br />And the last step is do add a markup: <br />
<div markdown="1">
{% highlight typescript %}
this._topPlaceholder.domElement.innerHTML = `
<div class="${styles.app}">
  <div class="ms-bgColor-themeDark ms-fontColor-white ${styles.header}">
    <i class="ms-Icon ms-Icon--Info" aria-hidden="true"></i> ${escape(strings.Message)}&amp;nbsp;
    <a href="${this._viewUrl}" target="_blank">${escape(strings.GoToList)}</a>
  </div>
</div>`;

{% endhighlight %}
</div>
Full code of <span class="code">_renderPlaceholder</span> looks like that: <br />
<div markdown="1">
{% highlight typescript %}
if (!this._overdueTasks || !this._overdueTasks.length) {
  return;
}

  if (!this._topPlaceholder) {
    this._topPlaceholder = this.context.placeholderProvider.tryCreateContent(PlaceholderName.Top, {
      onDispose: () => { }
    });
  }

  this._topPlaceholder.domElement.innerHTML = `
  <div class="${styles.app}">
<div class="ms-bgColor-themeDark ms-fontColor-white ${styles.header}">
>i aria-hidden="true" class="ms-Icon ms-Icon--Info"&amp;lgt;</i> ${escape(strings.Message)}&amp;nbsp;
      <a href="https://www.blogger.com/$%7Bthis._viewUrl%7D" target="_blank">${escape(strings.GoToList)}</a>
    </div>
</div>
`;
}

{% endhighlight %}
</div>
<h2>Deployment</h2>Deployment of any SharePoint Framework solution is done similarly to the deployment on SharePoint Add-ins: <br /><ul><li>Package the solution</li><li>Deploy the solution to App Catalog</li><li>Install the app to the specific site</li></ul>To package the solution you should run 2 Gulp tasks (note that you need to modify generated <span class="code">solution_folder/sharepoint/assets/elements.xml</span> file if you want to change the title of the Custom Action or provide correct properties): <br />
<div markdown="1">
{% highlight console %}
gulp bundle --ship
gulp package-solution --ship

{% endhighlight %}
</div>
The first one will compile and bundle all the code in Release mode (without debug information and with minification)<br />The second one will create .sppkg file in <span class="code">solution_folder/sharepoint</span> folder and also prepare all the assets to be deployed to some storage. It may be Office 365 CDN, Azure Storage or any other type of public or organization storage. Preparation of the storage is a large separate topic. You can read about Office 365 CDN <a href="https://dev.office.com/sharepoint/docs/spfx/extensions/get-started/hosting-extension-from-office365-cdn">here</a> and about Azure CDN <a href="https://dev.office.com/sharepoint/docs/spfx/web-parts/get-started/deploy-web-part-to-cdn">here</a>.<br />After the solution is packaged and the assets are deployed to some CDN the solution can be deployed (manually or using some custom routine) to App Catalog and later installed to some site.<br />By default, the application customizer will be available as soon as the app is installed to the site. If you need some custom logic that will optionally activate the Customizer you can use CSOM/JSOM for that: <ol><li>Delete <span class="code">elements.xml</span> in <span class="code">solution_folder/sharepoint/assets</span></li><li>Remove feature from <span class="code">package-solution.json</span> that references <span class="code">elements.xml</span></li><li>Deploy the solution to App Catalog and install on the site</li><li>Get Web CustomActions collection: 
<div markdown="1">
{% highlight csharp %}
var ctx = new ClientContext("web_url");
var web = ctx.Web;
var customActions = web.UserCustomActions;

{% endhighlight %}
</div>
</li><li>Add new CustomAction with correct ClientSideComponentId and ClientSideComponentProperties: var ca = customActions.Add(); ca.ClientSideComponentId = new Guid("customzier_id"); ca.ClientSideComponentProperties = "customizer_props"; ca.Title = "Application Customizer Title"; ca.Location = "ClientSideExtension.ApplicationCustomizer"; ca.Update(); web.Update(); ctx.ExecuteQuery(); </li></ol>Now the Application Customizer will be added to the specific site.<br />And that's it!<br />Now you should be ready to create your Application Customizers, debug them and deploy. You can get the code from <a href="https://github.com/SharePoint/sp-dev-fx-extensions/tree/master/samples/react-application-duetasks" target="_blank">GitHub repo</a><br /><br />Have fun! 