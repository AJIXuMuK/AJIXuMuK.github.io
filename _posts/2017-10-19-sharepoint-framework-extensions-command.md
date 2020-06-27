---
layout: post
title: 'SharePoint Framework Extensions: Command Set'
date: '2017-10-19T17:46:00.000-07:00'
author: Alex Terentiev
tags:
- SharePoint Online
- SPFx
- O365
- Command Set
- SharePoint Framework
- Office 365
- SharePoint Framework Extensions
- SharePoint
modified_time: '2017-10-19T17:46:13.350-07:00'
thumbnail: https://3.bp.blogspot.com/-g3Ag5aRiuK4/WePuJMgWoLI/AAAAAAAAAoc/OWJWyspDXrAXA8JJhuUAoRzyRriOUjcUwCLcBGAs/s72-c/command-set-yo.png
blogger_id: tag:blogger.com,1999:blog-3066084330774405472.post-1895606369274240157
blogger_orig_url: http://blog.aterentiev.com/2017/10/sharepoint-framework-extensions-command.html
---

 In this post I want to show you step-by-step implementation of SharePoint Framework Command Set Customizer.<br />Key features of the sample:<br /><ul><li>SPFx Command Set Customizer implementation</li><li>Current user permissions check</li><li>construction of url to work on specific item's permissions</li></ul>For those who doesn't want to read the post you can get it from&nbsp;<a href="https://github.com/AJIXuMuK/sp-dev-fx-extensions/tree/js-command-item-permissions/samples/js-command-item-permissions" target="_blank">here</a>. <br /><br /><a name='more'></a><h2>Prerequsites</h2>Before following the steps in this article, be sure to proceed next 3 steps: <br /><ul><li>Get Office Developer Tenant (you can get one for free by subscribing to&nbsp;<a href="http://dev.office.com/devprogram" target="_blank">Office 365 Developer Program</a>. It's an optional step as Extensions reached General Availability but it's good to have the tenant to be able to get preview versions in future. </li><li><a href="https://dev.office.com/sharepoint/docs/spfx/set-up-your-development-environment" target="_blank">Set up your development environment</a></li></ul><h2>General Information</h2><h3>Customizer Purpose</h3><ul><li>Insert custom commands to Lists’ command bar</li><li>Insert custom commands to List Items’ context menu (ECB)</li></ul><h3>Entry Points</h3><ul><li><span class="code">onInit()</span> – runs code before page DOM is ready. Perform needed setup here </li><li><span class="code">onListViewUpdated(IListViewCommandSetListViewUpdatedParameters)</span> – occurs whenever the application attempts to display it in the UI (for example, when user selects items in the list view) </li><li><span class="code">onExecute(IListViewCommandSetExecuteEventParameters)</span> – defines what happens when a command is executed </li></ul><h3>Helpful APIs</h3><ul><li><span class="code">this.properties</span> – properties that are passed to the customizer</li><li><span class="code">this.context.pageContext</span> – standard definitions for common SharePoint objects (site, web, user, list, etc.)</li><li><span class="code">this.context.httpClient, this.context.spHttpClient, this.context.graphHttpClient</span> – helpers to send http requests, http requests with SharePoint context and Microsoft Office Graph requests</li><li><span class="code">this.context.listView</span> – information about current List View</li><li><span class="code">event.itemId</span> – identifier of executed command (available in <span class="code">onExecute</span> method)</li><li><span class="code">event.selectedRows</span> - list of currently selected rows in the view. Available in <span class="code">onListViewUpdated</span> and <span class="code">onExecute</span> methods</li><li><span class="code">Command</span> - entity of the command. Contains command's editable properties (<span class="code">title</span>, <span class="code">visible</span>, etc.). The entity can be requested with <span class="code">this.tryGetCommand('COMMAND_ID')</span></li></ul><h2>Scaffolding the Project</h2>First step in any SPFx project is to "scaffold" it using Yeoman SharePoint Generator. "Scaffold" in more classic terms means generate or create from a template.<br />Here are the steps to create a project.<br /><ol><li>Open Command Prompt (Terminal in MacOS) or any Console tool</li><li>Create a new directory for the project in the location you want 
<div markdown="1">
{% highlight console %}
mkdir js-command-item-permissions

{% endhighlight %}
</div>
</li><li>Go to the created directory 
<div markdown="1">
{% highlight console %}
cd js-command-item-permissions

{% endhighlight %}
</div>
</li><li>Start scaffolding new project with Yeoman SharePoint Generator 
<div markdown="1">
{% highlight console %}
yo @microsoft/sharepoint

{% endhighlight %}
</div>
</li><li>When prompted <ul><li>Accept default solution name (js-command-item-permissions)</li><li>For baseline packages select SharePoint Online only (latest) as SPFx Extensions are not available for SharePoint 2016</li><li>Select Use the current folder</li><li>You can select either y or N for availability of your extension across tenant. It depends on how you're going to install it. If you use Feature Framework to add application customizer then this option will not work as it doesn't work with Feature Framework</li><li>Select Extension as a client-side component</li><li>Select List View Command Set as an extension type</li><li>Type ItemPermissions as a name</li><li>Accept default description</li></ul></li><li>Wait until Yeoman is installing all needed dependencies</li></ol><a href="https://3.bp.blogspot.com/-g3Ag5aRiuK4/WePuJMgWoLI/AAAAAAAAAoc/OWJWyspDXrAXA8JJhuUAoRzyRriOUjcUwCLcBGAs/s1600/command-set-yo.png" imageanchor="1" ><img border="0" src="https://3.bp.blogspot.com/-g3Ag5aRiuK4/WePuJMgWoLI/AAAAAAAAAoc/OWJWyspDXrAXA8JJhuUAoRzyRriOUjcUwCLcBGAs/s400/command-set-yo.png" width="400" height="230" data-original-width="1340" data-original-height="772" /></a>Congratulations! The project has been scaffolded! Now you can open the project in favorite IDE. <br /><b>Note: </b>this is a TypeScript project so select the IDE that supports TypeScript language. I'm using&nbsp;<a href="https://code.visualstudio.com/" target="_blank">Visual Studio Code</a>.<br />The project structure should look like the one on the picture below<br /><a href="https://1.bp.blogspot.com/-6v9YorceHIE/WePvW3gP8CI/AAAAAAAAAoo/g_31MXs3_dIa7xEdRrO3HhIMsjZ90oJUgCLcBGAs/s1600/proj-structure.png" imageanchor="1" ><img border="0" src="https://1.bp.blogspot.com/-6v9YorceHIE/WePvW3gP8CI/AAAAAAAAAoo/g_31MXs3_dIa7xEdRrO3HhIMsjZ90oJUgCLcBGAs/s400/proj-structure.png" width="282" height="400" data-original-width="657" data-original-height="933" /></a><br /><h2>Debugging the Extension using gulp serve and query string</h2>I decided to put this section at the beginning so you could run the code each time you want to check if everything works fine.<br />SharePoint Framework extensions cannot currently be tested using the local workbench, so you'll need to test and develop them directly against a live SharePoint Online site. You do not however need to deploy your customization to the app catalog to do this, which keeps the debugging experience simple and efficient. First, compile your code and host the compiled files from the local machine by running this command: <br />
<div markdown="1">
{% highlight console %}
gulp serve --nobrowser

{% endhighlight %}
</div>
<span class="code">--nobrowser</span> parameter tells to start the server but do not open workbench page in the browser. <br />If the code compiled without errors the server will be running on http://localhost:4321.<br />To test the extension<br /><ol><li>Go to SharePoint tenant</li><li>Open any list/document library</li><li>Append next query string parameters to the list's URL: 
<div markdown="1">
{% highlight javascript %}
?loadSPFX=true&amp;debugManifestsFile=https://localhost:4321/temp/manifests.js&amp;customActions={"fc875dc3-2180-4f1b-bc1c-a20d8ece961b":{"location":"ClientSideExtension.ListViewCommandSet"}}

{% endhighlight %}
</div>
Where: <ul><li><b>loadSPFX=true</b> ensures that the SharePoint Framework is loaded on the page. For performance reasons, the framework is not normally loaded unless at least one extension is registered. Since no components are registered yet, we must explicitly load the framework.</li><li><b>debugManifestsFile=&lt;URL&gt;</b> specifies that we want to load SPFx components that are being locally served. Normally the loader only looks for components in the App Catalog (for your deployed solution) and the SharePoint manifest server (for the system libraries).</li><li><b>customActions</b> simulates Custom Action. The Key here specifies the GUID of the extension that should be loaded - ID value is located in <span class="code">manifest.json</span> file of the extension. Location parameter should be  <ul><li><span class="code">ClientSideComponent.ListViewCommandSet</span> to add commands both to Command Bar and Item Context Menu</li><li><span class="code">ClientSideComponent.ListViewCommandSet.CommandBar</span> to add commands to Command Bar only</li><li><span class="code">ClientSideComponent.ListViewCommandSet.ContextMenu</span> to add commands to Item Context Menu only</li></ul>The properties parameter is an optional text string containing a JSON object that will be deserialized into <span class="code">this.properties</span> for your extension.</li></ul>The full URL should be similar to following (depending on tenant URL and List URL): 
<div markdown="1">
{% highlight javascript %}
https://terentiev.sharepoint.com/Lists/Slider%20Test/AllItems.aspx?loadSPFX=true&amp;debugManifestsFile=https://localhost:4321/temp/manifests.js&amp;customActions={"fc875dc3-2180-4f1b-bc1c-a20d8ece961b":{"location":"ClientSideExtension.ListViewCommandSet"}}

{% endhighlight %}
</div>
</li><li>Accept loading of Debug Manifests <br /><a href="https://2.bp.blogspot.com/-pz-7etinYJ0/WVwlqfDQDpI/AAAAAAAAAg0/mw96xs0nwKcVSQkmK2ZAnU_vCweqzkDwwCLcBGAs/s1600/allow-debug-scripts.png" imageanchor="1"><img border="0" data-original-height="563" data-original-width="1036" height="217" src="https://2.bp.blogspot.com/-pz-7etinYJ0/WVwlqfDQDpI/AAAAAAAAAg0/mw96xs0nwKcVSQkmK2ZAnU_vCweqzkDwwCLcBGAs/s400/allow-debug-scripts.png" width="400" /></a></li><li>You should buttons added to List's command bar<br /><a href="https://3.bp.blogspot.com/-wMrxPORTsqM/Wek5iYOcqnI/AAAAAAAAApI/EXdro-0zVVwfHHoQBxZ7sUrDy07JqF7AQCLcBGAs/s1600/Screen%2BShot%2B2017-10-19%2Bat%2B4.46.55%2BPM.png" imageanchor="1" ><img border="0" src="https://3.bp.blogspot.com/-wMrxPORTsqM/Wek5iYOcqnI/AAAAAAAAApI/EXdro-0zVVwfHHoQBxZ7sUrDy07JqF7AQCLcBGAs/s400/Screen%2BShot%2B2017-10-19%2Bat%2B4.46.55%2BPM.png" width="400" height="355" data-original-width="810" data-original-height="718" /></a></li></ol><h2>Configuring Command's Identifier and Title</h2>Next step is to change auto-generated identifiers and titles for the commands in the customizer.<br />By default, Yeoman generator adds 2 commands in the customizer. But in this example we need only one.<br />Commands definitions are stored in <span class="code">.manifest.json</span> file in the same location where the customizer's source code is located.<br />Default content of the file is shown below:<br /><a href="https://1.bp.blogspot.com/-C-Nzh-hvKQY/Wek9O49ehqI/AAAAAAAAApU/KAJjmVGZGjUcRbtxfm76pUO2eGuUzfVwACLcBGAs/s1600/Screen%2BShot%2B2017-10-19%2Bat%2B5.02.20%2BPM.png" imageanchor="1" ><img border="0" src="https://1.bp.blogspot.com/-C-Nzh-hvKQY/Wek9O49ehqI/AAAAAAAAApU/KAJjmVGZGjUcRbtxfm76pUO2eGuUzfVwACLcBGAs/s400/Screen%2BShot%2B2017-10-19%2Bat%2B5.02.20%2BPM.png" width="400" height="244" data-original-width="1600" data-original-height="974" /></a><br/>Let's remove default commands' definition and add command with Id <span class="code">ITEM_PERMISSIONS</span> and default title <span class="code">Set item permissions</span>.<br /><span class="code">iconImageUrl</span> may reference any image address that is available from the tenant. For this example let's just type <span class="code">"fake.png"</span><br /><b>Note:</b> for now there is no possibility to use font icons (for example, Office UI Fabric typography for Command Set commands' icons.<br/>Updated <span class="code">.manifest.json</span> file content should look like that: 
<div markdown="1">
{% highlight javascript %}

{
  "$schema": "https://dev.office.com/json-schemas/spfx/command-set-extension-manifest.schema.json",

  "id": "fc875dc3-2180-4f1b-bc1c-a20d8ece961b",
  "alias": "ItemPermissionsCommandSet",
  "componentType": "Extension",
  "extensionType": "ListViewCommandSet",

  // The "*" signifies that the version should be taken from the package.json
  "version": "*",
  "manifestVersion": 2,

  // If true, the component can only be installed on sites where Custom Script is allowed.
  // Components that allow authors to embed arbitrary script code should set this to true.
  // https://support.office.com/en-us/article/Turn-scripting-capabilities-on-or-off-1f2c515f-5d7e-448a-9fd7-835da935584f
  "requiresCustomScript": false,

  "items": {
    "ITEM_PERMISSIONS": {
      "title": { "default": "Set item permissions" },
      "iconImageUrl": "fake.png",
      "type": "command"
    }
  }
}

{% endhighlight %}
</div>
<h2>Configure command's UI behavior when ListView is refreshed</h2>As I mentioned above, there is a <span class="code">onListViewUpdated(IListViewCommandSetListViewUpdatedParameters)</span> event that occurs separately for each command (for example, a menu item) whenever a change happens in the List View, and the UI needs to be re-rendered. It can be used to determine if the command should be displayed and even its title.<br />In this example the command works with single item permissions. It means that it should be visible if 2 conditions were fulfilled: <ul><li>Current user has rights to manage permissions</li><li>Exactly one row is selected in the List View</li></ul>First thing to do is to get the command using <span class="code">tryGetCommand</span> method: 
<div markdown="1">
{% highlight typescript %}

const compareOneCommand: Command = this.tryGetCommand('ITEM_PERMISSIONS');

{% endhighlight %}
</div>
Needed permissions can be checked using next line of code: 
<div markdown="1">
{% highlight typescript %}

this.context.pageContext.list.permissions.hasPermission(SPPermission.managePermissions)

{% endhighlight %}
</div>
where <span class="code">SPPermission</span> is imported from <span class="code">@microsoft/sp-page-context</span><br />Finally, amount of selected rows could be received as <span class="code">event.selectedRows.length</span><br />Full code of <span class="code">onListViewUpdated</span> is listed below 
<div markdown="1">
{% highlight typescript %}

const compareOneCommand: Command = this.tryGetCommand('ITEM_PERMISSIONS');
if (compareOneCommand) {
  // This command should be hidden unless exactly one row is selected.
  compareOneCommand.visible = this.context.pageContext.list.permissions.hasPermission(SPPermission.managePermissions)
    && event.selectedRows.length === 1;
}

{% endhighlight %}
</div>
<h2>Executing the command</h2>The final part of the implementation is command's execution. <span class="code">onExecute</span> method is used to define the command's logic.<br />First this to do in this method is to check what command was executed (it is an optional step in scenario when a command set customizer has only one command definition, but necessary if there are 2 or more commands. That's why I would recommend to make this check in any command set): 
<div markdown="1">
{% highlight typescript %}

switch (event.itemId) {
  case 'ITEM_PERMISSIONS':
  // command's logic goes here

{% endhighlight %}
</div>
And in this example the only thing we need to do is to open a new window with item's permissions page. For that we need to open page <span class="code">/_layouts/15/user.aspx</span> with the next set of query parameters: <ul><li><span class="code">List</span> - list's ID. Braces should be "escaped" with %7B and %7D</li><li><span class="code">obj</span> - defines the object in the list that will be changed. It should contain list ID, item ID and LISTITEM word separated by comma </ul>The code to open a window with described url is listed below: 
<div markdown="1">
{% highlight typescript %}

const listId = this.context.pageContext.list.id;
window.open(`${this.context.pageContext.web.absoluteUrl}/_layouts/15/user.aspx?List=%7B${listId}%7D&obj=%7B${listId}%7D,${event.selectedRows[0].getValueByName('ID')},LISTITEM`, '_blank');

{% endhighlight %}
</div>
Full code of <span class="code">onExecute</span>: 
<div markdown="1">
{% highlight typescript %}

@override
public onExecute(event: IListViewCommandSetExecuteEventParameters): void {
  switch (event.itemId) {
    case 'ITEM_PERMISSIONS':
      const listId = this.context.pageContext.list.id;
      window.open(`${this.context.pageContext.web.absoluteUrl}/_layouts/15/user.aspx?List=%7B${listId}%7D&obj=%7B${listId}%7D,${event.selectedRows[0].getValueByName('ID')},LISTITEM`, '_blank');

      break;
    default:
      throw new Error('Unknown command');
  }
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
var list = web.Lists.GetByTitle('My List');
var customActions = list.UserCustomActions;

{% endhighlight %}
</div>
</li><li>Add new CustomAction with correct ClientSideComponentId and ClientSideComponentProperties: var ca = customActions.Add(); ca.ClientSideComponentId = new Guid("customzier_id"); ca.ClientSideComponentProperties = "customizer_props"; ca.Title = "List View Command Set Customizer Title"; ca.Location = "ClientSideExtension.ListViewCommandSet"; ca.Update(); list.Update(); ctx.ExecuteQuery(); </li></ol>Now the List View Command Set Customizer will be added to the specific list.<br />And that's it!<br />Now you should be ready to create your Application Customizers, debug them and deploy. You can get the code from <a href="https://github.com/AJIXuMuK/sp-dev-fx-extensions/tree/js-command-item-permissions/samples/js-command-item-permissions" target="_blank">GitHub repo</a><br /><br />Have fun!  