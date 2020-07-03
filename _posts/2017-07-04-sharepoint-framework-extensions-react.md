---
layout: post
title: 'SharePoint Framework Extensions: React Slider Field Customizer'
date: '2017-07-04T17:24:00.002-07:00'
author: Alex Terentiev
tags:
- SharePoint Online
- npm
- Gulp
- Node.js
- SharePoint Framework
- Office 365
- SharePoint Framework Extensions
- JavaScript
- Field Customizer
- SPFx
- Yeoman
- generator-sharepoint
- O365
- Future of SharePoint
modified_time: '2017-10-02T11:39:11.631-07:00'
thumbnail: assets/images/posts/2017/2017-07-04-1.png
featured_image: assets/images/posts/2017/2017-07-04-1.png
blogger_id: tag:blogger.com,1999:blog-3066084330774405472.post-5343379554607515323
blogger_orig_url: http://blog.aterentiev.com/2017/07/sharepoint-framework-extensions-react.html
---

In this post I want to show you step-by-step implementation of SharePoint Framework Field Customizer - React Slider Customizer.<br /><b>UPDATE:</b> on 9/25/2017 SPFx Extensions were GA-ed. The post is updated to contain the latest version of API.<br />Key features of the sample: <ul>    <li>SPFx Field Customizer implementation</li>    <li>Usage of <a href="https://dev.office.com/fabric" target="_blank">Office UI Fabric</a> controls (in particular - <a href="https://dev.office.com/fabric#/components/slider" target="_blank">Slider Control</a>)</li>    <li>Current user's permissions check</li>    <li>Inline editing</li>    <li>Usage of&nbsp;<a href="https://github.com/SharePoint/PnP-JS-Core" target="_blank">SharePoint PnP JavaScript Core Library</a></li></ul>For those who doesn't want to read the post but need the code - you can get it from&nbsp;<a href="https://github.com/SharePoint/sp-dev-fx-extensions/tree/master/samples/react-field-slider" target="_blank">here</a>. <a name='more'></a><h2>    Prerequisites </h2>    Before following the steps in this article, be sure to proceed next 2 steps: <ul>    <li>Get Office Developer Tenant (you can get one for free by subscribing to&nbsp;<a href="http://dev.office.com/devprogram" target="_blank">Office 365 Developer Program</a>. It's an optional step as Extensions reached General Availability but it's good to have the tenant to be able to get preview versions in future.</li>    <li><a href="https://dev.office.com/sharepoint/docs/spfx/set-up-your-development-environment" target="_blank">Set up your development environment</a></li></ul><h2>    Scaffolding the Project </h2>    First step in any SPFx project is to "scaffold" it using Yeoman SharePoint Generator. "Scaffold" in more classic terms means generate or create from a template.<br />    Here are the steps to create a project. <ol>    <li>Open Command Prompt (Terminal in MacOS) or any Console tool</li>    <li>        Create a new directory for the project in the location you want 
<div markdown="1">
{% highlight console %}
mkdir react-field-slider

{% endhighlight %}
</div>
    </li>    <li>        Go to the created directory 
<div markdown="1">
{% highlight console %}
cd react-field-slider

{% endhighlight %}
</div>
    </li>    <li>        Start scaffolding new project with Yeoman SharePoint Generator 
<div markdown="1">
{% highlight console %}
yo @microsoft/sharepoint

{% endhighlight %}
</div>
    </li>    <li>        When prompted         <ul>            <li>Accept default solution name (react-field-slider)</li><li>For baseline packages select SharePoint Online only (latest) as SPFx Extensions are not available for SharePoint 2016</li><li>Select Use the current folder</li><li>You can select either y or N for availability of your extension across tenant. It depends on how you're going to install it. If you use Feature Framework to add application customizer then this option will not work as it doesn't work with Feature Framework</li>            <li>Select Extension as a client-side component</li>            <li>Select Field Customizer as an extension type</li>            <li>Type Slider as a name</li>            <li>Accept default description</li>            <li>Select React as a framework</li>        </ul>    </li>    <li>Wait until Yeoman is installing all needed dependencies</li></ol><img border="0" src="{{site.baseurl}}/assets/images/posts/2017/2017-07-04-1.png" /><br />    Congratulations! The project has been scaffolded! <br />    Now you can open the project in favorite IDE.<br /><b>Note: </b>this is a TypeScript project so select the IDE that supports TypeScript language. I'm using&nbsp;<a href="https://code.visualstudio.com/" target="_blank">Visual Studio Code</a>.<br />    The project structure should look like the one on the picture below<br /><div style="display: block; text-align: left;"><img border="0" src="{{site.baseurl}}/assets/images/posts/2017/2017-07-04-2.png" /></div><h2>    Debugging the Extension using gulp serve and query string </h2>    I decided to put this section at the beginning so you could run the code each time you want to check if everything works fine.<br />    SharePoint Framework extensions cannot currently be tested using the local workbench, so you'll need to test and develop them directly against a live SharePoint Online site. You do not however need to deploy your customization to the app catalog to do this, which keeps the debugging experience simple and efficient.     First, compile your code and host the compiled files from the local machine by running this command: <br />
<div markdown="1">
{% highlight console %}
gulp serve --nobrowser

{% endhighlight %}
</div>
    --nobrowser parameter tells to start the server but do not open workbench page in the browser. <br />    If the code compiled without errors the server will be running on http://localhost:4321.<br />    To test the extension<br /><ol>    <li>Go to SharePoint tenant's list</li>    <li>Open any list (or create one) that contains a column to be customized. In this sample Percent column will be customized</li>    <li>        Append next query string parameters to the list's URL: 
<div markdown="1">
{% highlight javascript %}
?loadSPFX=true&amp;debugManifestsFile=https://localhost:4321/temp/manifests.js&amp;fieldCustomizers={"Percent":{"id":"f2f6825c-fd37-43f7-a99c-5fe6b39dd7fd"}}

{% endhighlight %}
</div>
        Where:         <ul>            <li><b>loadSPFX=true</b> ensures that the SharePoint Framework is loaded on the page. For performance reasons, the framework is not normally loaded unless at least one extension is registered. Since no components are registered yet, we must explicitly load the framework.</li>            <li><b>debugManifestsFile=&lt;URL&gt;</b> specifies that we want to load SPFx components that are being locally served. Normally the loader only looks for components in the App Catalog (for your deployed solution) and the SharePoint manifest server (for the system libraries).</li>            <li><b>fieldCustomizers</b> indicates which fields in your list should have their rendering controlled by the Field Customizer. The ID parameter specifies the GUID of the extension that should be used to control the rendering of the field - ID value is located in manifest.json file of the extension. The properties parameter is an optional text string containing a JSON object that will be deserialized into this.properties for your extension (in current example properties parameter is not set).</li>        </ul>        The full URL should be similar to following (depending on tenant URL and List URL): 
<div markdown="1">
{% highlight javascript %}
https://terentiev.sharepoint.com/Lists/Slider%20Test/AllItems.aspx?loadSPFX=true&amp;debugManifestsFile=https://localhost:4321/temp/manifests.js&amp;fieldCustomizers={"Percent":{"id":"f2f6825c-fd37-43f7-a99c-5fe6b39dd7fd"}}

{% endhighlight %}
</div>
    </li>    <li>        Accept loading of Debug Manifests <br />        <img border="0" src="{{site.baseurl}}/assets/images/posts/2017/2017-07-04-3.png" />    </li>    <li>        The Percent column cells should be rendered using the Field Customizer         <img border="0" src="{{site.baseurl}}/assets/images/posts/2017/2017-07-04-4.png" /></li></ol><h2>    Adding Additional Libraries </h2>    Next step is to add external libraries (modules) that will be used in the project. For this sample the libraries are Office UI Fabric React and SharePoint PnP JS Core Library.<br />    Office UI Fabric is included automatically because React framework was selected as a basement of the extensions.<br />    PnP JS Core can be added using npm package manager:<br /><ol>    <li>In Command Prompt (Terminal) go to project directory.</li>    <li>        Install the PnP package 
<div markdown="1">
{% highlight console %}
npm install sp-pnp-js --save

{% endhighlight %}
</div>
    </li></ol>    Now the module can be imported to any file in the project<br />
<div markdown="1">
{% highlight typescript %}
import pnp from 'sp-pnp-js';

{% endhighlight %}
</div>
<h2>    Implementing Slider Field Customizer Control </h2>    The logic for the customizer is located in 2 main files: SliderFieldCustomizer.ts and components/Slider.tsx<br />    SliderFieldCustomizer class is the entry point of the customization. It provides 3 main methods to implement custom logic: <ol>    <li><span class="code">onInit&nbsp;</span>method allows to run some initialization code and ensure that the execution of this code is finished before rendering.</li>    <li><span class="code">onRenderCell&nbsp;</span>method is an entry point for actual rendering. You have access to cell's div to render your content. Also, there is a bunch of helpful objects inside&nbsp;<span class="code">this.properties</span>&nbsp;property and&nbsp;<span class="code">event</span>&nbsp;object (such as field value, field information, List View properties, etc.).</li>    <li><span class="code">onDisposeCell&nbsp;</span>method allows to release all used resources.</li></ol>    Slider class is a React component that should contain the html markup to be rendered in the cell. By default it renders formatted cell's value:<br /><img border="0" src="{{site.baseurl}}/assets/images/posts/2017/2017-07-04-5.png" /><br />    Let's change the Slider.tsx to display a Office UI Fabric Slider control.<br />    First, import Slider component from Office UI Fabric Slider<br />
<div markdown="1">
{% highlight typescript %}
import { Slider as ReactSlider } from 'office-ui-fabric-react';

{% endhighlight %}
</div>
    (named import is used as the control itself has name Slider). <br />    Next step is to modify Props and State objects to meet next conditions:<br /><ol>    <li>Props object should contain cell's value (nullable)</li>    <li>Props object should contain handler to process Slider value change (optional)</li>    <li>Props object should contain a flag to enable\disable the Slider based on user permissions</li>    <li>Props object should contain list item id to pass it to value change handler</li>    <li>State object should contain current value</li></ol>
<div markdown="1">
{% highlight typescript %}
export interface ISliderProps {
  value: string;
  id: string;
  disabled: boolean;
  onChange: (value: number, id: string) => void;
}
export interface ISliderState {
  value?: number;
}

{% endhighlight %}
</div>
    In class constructor initialize the control's state based on the value from props:<br />
<div markdown="1">
{% highlight typescript %}
constructor(props: ISliderProps, state: ISliderState) {
  super(props, state);
  const intVal = parseInt(props.value);
  this.state = {
    value: isNaN(intVal) ? null : intVal
};

{% endhighlight %}
</div>
    Now let's render the slider control in onRender method based on the parameters from Props and State (I've hardcoded min value to 0 and max value to 100 to simplify the example. In real world these values should be changed based on business logic). <br />
<div markdown="1">
{% highlight typescript %}
@override
public render(): React.ReactElement<{}> {
  return (
    <div classname="{styles.cell}">
{this.state.value &amp;&amp; // we're not rendering the slider if there is no value in the cell
      (
    <reactslider change="" current="" disabled="{this.props.disabled}" handler="" internal="" max="{100}" onchange="{this.onChange.bind(this)}" value=""> // disabled flag is based on user's permissions (see SliderFieldCustomizer class)
      )}
    </reactslider></div>
);
}
/**
 * value change internal handler
 */
private onChange(value: number): void {
  if (this.props.onChange) // we need to call external handler here
    this.props.onChange(value, this.props.id);
}

{% endhighlight %}
</div>
    Let's return to SliderFieldCustomizer class and modify it to pass correct props to Slider control and to handle value changes.<br />    Add <span class="code">onSliderValueChanged</span> empty method to use it as a value change handler (we'll add the code later). <br />
<div markdown="1">
{% highlight typescript %}
private onSliderValueChanged(value: number, id: string): void {
  // awesome code goes here
};

{% endhighlight %}
</div>
    Modify <span class="code">onRenderCell</span> method     to get list item id: <br />
<div markdown="1">
{% highlight typescript %}
const id: string = event.row.getValueByName('ID').toString();

{% endhighlight %}
</div>
    to check if user has permissions to edit items in the list: <br />
<div markdown="1">
{% highlight typescript %}
const hasPermissions: boolean = this.context.pageContext.list.permissions.hasPermission(SPPermission.editListItems);

{% endhighlight %}
</div>
    and to pass all the parameters to the Slider component: <br />
<div markdown="1">
{% highlight typescript %}
const slider: React.ReactElement<{}> =
  React.createElement(Slider, { value: value, id: id, disabled: !hasPermissions, onChange: this.onSliderValueChanged.bind(this) } as ISliderProps);

{% endhighlight %}
</div>
    Now Percent column should look like this: <img border="0" src="{{site.baseurl}}/assets/images/posts/2017/2017-07-04-6.png" /><br />    And if a user doesn't have permissions to edit items the sliders will be disabled and grayed out:<br /><img border="0" src="{{site.baseurl}}/assets/images/posts/2017/2017-07-04-7.png" /><br/>    The last part of the implementation is to save values' changes to the back-end using PnP JS Core Library.<br />    Go to <span class="code">onChange</span> handler (<span class="code">onSliderValueChanged</span> method) and add the code to update the value: <br />
<div markdown="1">
{% highlight typescript %}
let updateObj: any = {}; // object-parameter for update call
updateObj[this.context.field.internalName] = value; // the value from the slider
pnp.sp.web.lists
  .getByTitle(this.context.pageContext.list.title) // getting the list
  .items.getById(parseInt(id)) // getting the item
  .update(updateObj) // updating the item
  .then((result: ItemUpdateResult): void => {
    console.log(`Item with ID: ${id} successfully updated`);
  }, (error: any): void => {
    console.log('Loading latest item failed with error: ' + error);
  });

{% endhighlight %}
</div>
    This code will work but it has one major flaw: update request will be sent each time a user drags the slider. It might lead to multiple requests per second. It's much better to wait until a user stops interaction with the slider and send single request with the latest value.<br />    Let's use window.setTimeout to implement such logic. <br />
<div markdown="1">
{% highlight typescript %}
private onSliderValueChanged(value: number, id: string): void {
  if (this._timerId !== -1)
    clearTimeout(this._timerId);
  this._timerId = setTimeout(() => {
    let updateObj: any = {};
    updateObj[this.context.field.internalName] = value;
    pnp.sp.web.lists
      .getByTitle(this.context.pageContext.list.title) // getting the list
      .items.getById(parseInt(id)) // getting the item
      .update(updateObj) // updating the item
        .then((result: ItemUpdateResult): void => {
          console.log(`Item with ID: ${id} successfully updated`);
        }, (error: any): void => {
          console.log('Loading latest item failed with error: ' + error);
        });
  }, 1000);
}

{% endhighlight %}
</div>
    Now the request will be sent 1 second after last value change. <br />    That's it! You can get the code from <a href="https://github.com/SharePoint/sp-dev-fx-extensions/tree/master/samples/react-field-slider" target="_blank">GitHub repo</a>.<br /><br />    Have fun! 