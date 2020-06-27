---
layout: post
title: Connected SharePoint Framework Web Parts Using Dynamic Data
date: '2019-03-23T16:11:00.000-07:00'
author: Alex Terentiev
tags:
- SharePoint Online
- Client Side Web Part
- SPFx
- Client Web Part
- Office Development
- O365
- SharePoint Framework
- Dynamic Data
- Office 365
- SharePoint Development
- SharePoint
modified_time: '2019-03-23T16:19:28.471-07:00'
blogger_id: tag:blogger.com,1999:blog-3066084330774405472.post-5728944740138321792
blogger_orig_url: http://blog.aterentiev.com/2019/03/connected-sharepoint-framework-web.html
---

Using the dynamic data capability, you can connect SharePoint Framework client-side web parts and extensions to each other and exchange information between the components.<br /><a href="https://docs.microsoft.com/en-us/sharepoint/dev/spfx/dynamic-data" target="_blank">Official documentation</a> contains great example on how to connect web parts using <span class="code">DynamicProperty</span> web part properties that can consume values from other web part, or, generally, data source.<br />In my case I want to connect web parts in "old-school" way - with events firing and handling. Moreover, I want "receiver" web part to handle event from multiple "senders".<br />This blog post is based on Page Sections Navigation sample from <a href="https://github.com/SharePoint/sp-dev-fx-webparts/tree/dev/samples/react-page-sections-navigation" target="_blank">SPFx web parts samples</a> repository.<br /><a name='more'></a>So, the initial idea is to create a solution that allows to render page sections navigation on SharePoint modern page.<br />If you think of such a solution one of possible implementations would be to have "master" web part that renders the navigation, and multiple "anchor" web parts that can be added by a user anywhere on the page.<br />"Master" web part should know about all the anchors, their positions and titles to render navigation and handle navigation events.<br />And this is actually doable using dynamic data capability.<br />Let's start with "Anchor" web part implementation. I'll be partially skipping details of code to focus attention on Dynamic Data features and APIs.<br />As I mentioned above, "anchor" web part should notify "master" web part about its title and position (or DOM element instead of position). To do that we need to do few things: <ul><li>mark "anchor" web part as a "data source" - implement <span class="code"> IDynamicDataCallables</span> interface and two methods from this interface: <ul><li><span class="code">getPropertyDefinitions</span> to return list of dynamic data properties</li><li><span class="code">getPropertyValue</span> to get the value of specified property</li></ul></li><li>register the web part as a data source</li><li>notify the engine if any of dynamic properties has been changed</li></ul>There are two ways to implement dynamic properties for the "anchor" web part: either implement two properties: <span class="code">title</span> and <span class="code">domElement</span>, or combine them in single object. I selected the second approach.<br />First, let's declare <span class="code">IAnchorItem</span> interface: 
<div markdown="1">
{% highlight typescript %}

/**
 * Anchor interface to be transferred to the "master" web part
 */
export interface IAnchorItem {
    /**
     * Title
     */
    title?: string;
    /**
     * DOM element
     */
    domElement?: HTMLElement;
}

{% endhighlight %}
</div>
Next, implement the <span class="code">IDynamicDataCallables</span> interface: 
<div markdown="1">
{% highlight typescript %}

import { IDynamicDataPropertyDefinition, IDynamicDataCallables, IDynamicDataSource } from '@microsoft/sp-dynamic-data';
export default class PageSectionsNavigationAnchorWebPart extends BaseClientSideWebPart<IPageSectionsNavigationAnchorWebPartProps> implements IDynamicDataCallables {
  // ...
  // anchor data object related to the current web part
  private _anchor: IAnchorItem;
  /**
   * implementation of getPropertyDefinitions from IDynamicDataCallables
   */
  public getPropertyDefinitions(): ReadonlyArray<IDynamicDataPropertyDefinition> {
    return [{
      id: 'anchor',
      title: 'Anchor'
    }];
  }

  /**
   * implementation of getPropertyValue from IDynamicDataCallables
   * @param propertyId property Id
   */
  public getPropertyValue(propertyId: string): IAnchorItem {
    switch (propertyId) {
      case 'anchor':
        return this._anchor;
    }

    throw new Error('Bad property id');
  }
}

{% endhighlight %}
</div>
Next step is to register the web part as a data source using <span class="code">this.context.dynamicDataSourceManager.initializeSource</span>. Let's do that in <span class="code">onInit</span> method: 
<div markdown="1">
{% highlight typescript %}

  protected onInit(): Promise<void> {
    // registering current web part as a data source
    this.context.dynamicDataSourceManager.initializeSource(this);
  }

{% endhighlight %}
</div>
Now we can notify subscribers every time when title or DOM element has been changed using <span class="code">this.context.dynamicDataSourceManager.notifyPropertyChanged</span> method.<br />In provided sample I'm doing that based on events fired from React component: 
<div markdown="1">
{% highlight typescript %}

  public render(): void {
    //...
    const element: React.ReactElement<IPageSectionsNavigationAnchorProps> = React.createElement(
      PageSectionsNavigationAnchor,
      {
        //...
        updateProperty: (title => {
          this._anchor.title = this.properties.title = title;
          // notifying that title has been changed
          this.context.dynamicDataSourceManager.notifyPropertyChanged('anchor');
        }),
        anchorElRef: (el => {
          // notifying subscribers that the anchor component has been rendered
          this._anchor.domElement = el;
          this.context.dynamicDataSourceManager.notifyPropertyChanged('anchor');
        }),
        navPosition: position
      }
    );

    ReactDom.render(element, this.domElement);

  }

{% endhighlight %}
</div>
<br />Now let's work on "master" web part.<br />In this web part we need to collect all "anchor" data sources (web parts registered as data sources) from the page and subscribe to the changes of <span class="code">anchor</span> property. Two things to consider here: <ol><li>The page can contain data sources other than "anchors". These data sources should not be processed by the "master" web part.</li><li>"anchors" data sources can be dynamically added/removed based on user's interaction.</li></ol>Currently, there is no tutorial on how to correctly work with data sources. But there is an <a href="https://docs.microsoft.com/en-us/javascript/api/overview/sharepoint?view=sp-typescript-latest" target="_blank">SharePoint Framework API Reference</a> that will help us! And, in particular, <a href="https://docs.microsoft.com/en-us/javascript/api/sp-component-base/dynamicdataprovider?view=sp-typescript-latest" target="_blank">DynamicDataProvider</a> and <a href="https://docs.microsoft.com/en-us/javascript/api/sp-component-base/dynamicdatasourcemanager?view=sp-typescript-latest" target="_blank">DynamicDataSourceManager</a> classes.<br />The first one is exposed as <span class="code">this.context.dynamicDataProvider</span> in web parts and allows to get all data sources from the page as well as register for the change of available data sources.<br />
<div markdown="1">
{% highlight typescript %}

/**
* Registers a callback to an event that raises when the list of available Dynamic Data Sources is updated.
*
* @param callback - Function to execute when the sources are updated.
*/
registerAvailableSourcesChanged(callback: () => void): void;
/**
* Returns a list with all available Dynamic Data Sources.
*
* @returns Read-only array with all available sources.
*/
getAvailableSources(): ReadonlyArray<IDynamicDataSource>;

{% endhighlight %}
</div>
The second one is exposed as <span class="code">this.context.dynamicDataSourceManager</span> in web parts and allows to register on properties' changes of specific data sources.<br />Knowing the above API we can easily collect "anchor" data sources dynamically and register on "anchor" property change.<br />First, let's add a field that will store current "anchors" on the page: 
<div markdown="1">
{% highlight typescript %}

// "Anchor" data sources
private _dataSources: IDynamicDataSource[] = [];

{% endhighlight %}
</div>
Next, let's create the method that will iterate through page's data sources, update <span class="code">_dataSources</span> collection, and subscribe on "anchor" property change event: 
<div markdown="1">
{% highlight typescript %}

/**
 * Initializes collection of "Anchor" data soures based on collection of existing page's data sources
 */
private _initDataSources() {
  // all data sources on the page
  const availableDataSources = this.context.dynamicDataProvider.getAvailableSources();

  if (availableDataSources && availableDataSources.length) {
    // "Anchor" data sources cached in the web part from prev call
    const dataSources = this._dataSources;
    //
    // removing deleted data sources if any
    //
    const availableDataSourcesIds = availableDataSources.map(ds => ds.id);
    for (let i = 0, len = dataSources.length; i < len; i++) {
      let dataSource = dataSources[i];
      if (availableDataSourcesIds.indexOf(dataSource.id) == -1) {
        dataSources.splice(i, 1);
        try {
          this.context.dynamicDataProvider.unregisterPropertyChanged(dataSource.id, 'anchor', this._onAnchorChanged);
        }
        catch (err) { }
        i--;
        len--;
      }
    }

    //
    // adding new data sources
    //
    for (let i = 0, len = availableDataSources.length; i < len; i++) {
      let dataSource = availableDataSources[i];
      if (!dataSource.getPropertyDefinitions().filter(pd => pd.id === 'anchor').length) {
        continue; // we don't need data sources other than anchors
      }
      if (!dataSources || !dataSources.filter(ds => ds.id === dataSource.id).length) {
        dataSources.push(dataSource);
        this.context.dynamicDataProvider.registerPropertyChanged(dataSource.id, 'anchor', this._onAnchorChanged);
      }
    }
  }
}

{% endhighlight %}
</div>
The last part is to call this method when the "master" web part is being initialize and every time when data sources are being changed. 
<div markdown="1">
{% highlight typescript %}

protected onInit(): Promise<void> {    
  // getting data sources that have already been added on the page
  this._initDataSources();
  // registering for changes in available datasources
  this.context.dynamicDataProvider.registerAvailableSourcesChanged(this._initDataSources.bind(this, true));
}

{% endhighlight %}
</div>
<br /><h2>Conclusion</h2>Connecting web part using SPFx Dynamic Data is simple. Here are configuration/implementation steps: <ul><li>In sender: <ul><li>Implement <span class="code">IDynamicDataCallables</span> interface in a web part with two methods: <span class="code">getPropertyDefinitions</span> and <span class="code">getPropertyValue</span></li><li>Register a web part as a Data Source using <span class="code">this.context.dynamicDataSourceManager.initializeSource</span></li><li>Notify subscriber that some of properties have been changed using <span class="code">this.context.dynamicDataSourceManager.notifyPropertyChanged</span></li></ul></li><li>In receiver: <ul><li>Use <span class="code">this.context.dynamicDataProvider.registerAvailableSourcesChanged</span> and <span class="code">this.context.dynamicDataProvider.getAvailableSources</span> to iterate through available page's data source and select the ones to work with</li><li>Use <span class="code">this.context.dynamicDataProvider.registerPropertyChanged</span> to register event handlers for specific data sources and properties</li></ul></li></ul>The example from this post processes single event from one type of data source. But you can use the same API to process different properties, data sources, and also implement circle connections between web parts. <br /><br />That's all for today!<br />Have fun! 