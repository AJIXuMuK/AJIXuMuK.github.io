---
layout: post
title: 'SPFx. Client Web Part Property Pane Dependent Properties. Part I: Preparation.'
date: '2016-10-11T15:10:00.001-07:00'
author: Alex Terentiev
tags:
- JavaScript
- SharePoint Online
- SPFx
- Client Web Part
- SharePoint Framework
- Future of SharePoint
- SharePoint
modified_time: '2016-10-28T12:52:01.551-07:00'
thumbnail: /assets/images/posts/2016/2016-10-11-1.png
featured_image: /assets/images/posts/2016/2016-10-11-1.png
blogger_id: tag:blogger.com,1999:blog-3066084330774405472.post-4634049440308298512
blogger_orig_url: http://blog.aterentiev.com/2016/10/spfx-client-web-part-property-pane.html
---

This is the first post of three that explain how to add dependent properties to Client Web Part Property Pane.<br />As an example of such properties I decided to use Lists dropdown and Views dropdown. Views dropdown is populated based on selected List.<br />As a result of these three posts we'll have a fully working web part with ability to select List View in Property Pane:<br /><div class="separator" style="clear: both; text-align: center;"><img border="0" src="{{site.baseurl}}/assets/images/posts/2016/2016-10-11-1.png" /></div><br />Here is a content of 3 posts: <br /><ol><li>SPFx. Client Web Part Property Pane Dependent Properties. Part I: Preparation. (current post)</li><li><a href="http://tricky-sharepoint.blogspot.com/2016/10/spfx-client-web-part-property-pane_26.html" target="_blank">SPFx. Client Web Part Property Pane Dependent Properties. Part II: Dependent Properties Control.</a></li><li><a href="http://tricky-sharepoint.blogspot.com/2016/10/spfx-client-web-part-property-pane_27.html" target="_blank">SPFx. Client Web Part Property Pane Dependent Properties. Part III: Styling Dependent Properties Control as Office UI Fabric dropdowns. Knockout version.</a></li></ol><div>All the code is available on GitHub (<a href="https://github.com/AJIXuMuK/SPFx/tree/master/dep-props-custom-class" target="_blank">https://github.com/AJIXuMuK/SPFx/tree/master/dep-props-custom-class</a>)<br />I'm using Visual Studio Code IDE for the tutorial.</div><div><a name='more'></a><br />So let's start with preparation.<br /><br />This post describes how to set up your environment and implement code to request data from SharePoint or Mock Store based on your environment (local workbench or SharePoint Online).<br /><br /><h3>Setting Up Environment</h3>If it's your first touch of SharePoint Framework (SPFx) then you'll need to set up your environment (O365 and local) as it is described in Office Dev Center:</div><div><a href="http://dev.office.com/sharepoint/docs/spfx/set-up-your-developer-tenant" target="_blank">Set up your Office 365 developer tenant</a></div><div><a href="http://dev.office.com/sharepoint/docs/spfx/set-up-your-development-environment" target="_blank">Set up your development environment</a></div><div><br /><h3>Scaffolding Web Part Project</h3>After setting up is done you can start with your web part creation. The process is described in&nbsp;<a href="http://dev.office.com/sharepoint/docs/spfx/web-parts/get-started/build-a-hello-world-web-part" target="_blank">Build your first web part</a>.</div><div>In brief, you need to execute multiple command line operations:</div><div><br /></div>Create a new directory in location you want:<br />
<div markdown="1">
{% highlight console %}
mkdir dep-props-webpart

{% endhighlight %}
</div>
<div>Go to created directory:<br />
<div markdown="1">
{% highlight console %}
cd dep-props-webpart

{% endhighlight %}
</div>
"Scaffold" your project with Yeoman generator:<br />
<div markdown="1">
{% highlight console %}
yo @microsoft/sharepoint

{% endhighlight %}
</div>
Select name, description and base framework for the web part (I'm not using any default framework in this example. I tried to select knockout but it was not compiling).<br />Wait for a while until all the dependencies will be downloaded.<br />Play with your Hello World web part using Gulp:<br />
<div markdown="1">
{% highlight console %}
gulp serve

{% endhighlight %}
</div>
If you're developing on Mac you'll probably need to run<br />
<div markdown="1">
{% highlight console %}
sudo gulp serve

{% endhighlight %}
</div>
It will run local server and SharePoint workbench for testing your web parts locally.<br /><br /><h3>Creating Data Helpers</h3></div><div>Basically, all the code of getting data is described in&nbsp;<a href="http://dev.office.com/sharepoint/docs/spfx/web-parts/get-started/connect-to-sharepoint" target="_blank">Connect to SharePoint</a>.</div><div>In brief, we need to implement retrieving some mock data for local environment (when we test our web part on local server created by <span style="font-family: &quot;courier new&quot; , &quot;courier&quot; , monospace;">gulp serve</span><span style="font-family: inherit;">) and retrieving SharePoint data.</span></div><div><span style="font-family: inherit;">We can use </span><span style="font-family: &quot;courier new&quot; , &quot;courier&quot; , monospace;">IWebPartContext.environment.type</span><span style="font-family: inherit;">&nbsp;property to understand what data to retrieve.</span></div><div><span style="font-family: inherit;">I don't like the idea of putting all the logic into the web part class itself. That's why I propose to create mock data helper, SharePoint data helper and data helpers factory to created needed helper based on current environment.</span></div><div><span style="font-family: inherit;">First, let's create all base entities we'll use in our app. (I created a subfolder named common in web part folder and </span><span style="font-family: &quot;courier new&quot; , &quot;courier&quot; , monospace;">SPEntities.ts</span><span style="font-family: inherit;">&nbsp;file in it.)</span></div><div><span style="font-family: inherit;"><br /></span></div><div><span style="font-family: &quot;courier new&quot; , &quot;courier&quot; , monospace;">ISPList</span><span style="font-family: inherit;"> - represents SharePoint List object with Title and Id</span></div>
<div markdown="1">
{% highlight typescript %}
/**
 * Represents SharePoint List object
 */
export interface ISPList {
  Title: string;
  Id: string;
}

{% endhighlight %}
</div>
<div><span style="font-family: inherit;"><br /></span></div><div><span style="font-family: &quot;courier new&quot; , &quot;courier&quot; , monospace;">ISPLists</span><span style="font-family: inherit;">&nbsp;- represents SharePoint REST service response for /_api/web/lists service call</span><br />
<div markdown="1">
{% highlight typescript %}
/**
 * Represents SharePoint REST service response for /_api/web/lists service call
 */
export interface ISPLists {
  value: ISPList[];
}

{% endhighlight %}
</div>
<span style="font-family: inherit;"><br /></span><span style="font-family: &quot;courier new&quot; , &quot;courier&quot; , monospace;">ISPView</span><span style="font-family: inherit;">&nbsp;- represents SharePoint View object with Title, Id and ListId</span><br />
<div markdown="1">
{% highlight typescript %}
/**
 * Represents SharePoint View object
 */
export interface ISPView {
  Title: string;
  Id: string;
  ListId: string;
}

{% endhighlight %}
</div>
<span style="font-family: inherit;"><br /></span><span style="font-family: &quot;courier new&quot; , &quot;courier&quot; , monospace;">ISPViews</span><span style="font-family: inherit;">&nbsp;- represents SharePoint REST service response for /_api/web/lists('id')/views service call</span><br />
<div markdown="1">
{% highlight typescript %}
/**
 * Represents SharePoint REST service response for /_api/web/lists('id')/views service call
 */
export interface ISPViews {
  value: ISPView[];
}

{% endhighlight %}
</div>
<br />Now we can start with data helpers. Let's create a separate subfolder for them and name it data-helpers.<br />First of all, we need some base interface that can be used to abstract of current data helper. What methods do we need? Well, not so much: a method to get lists and a method to get views of the list. Let's create such interface in a file named <span style="font-family: &quot;courier new&quot; , &quot;courier&quot; , monospace;">DataHelperBase.ts</span><br />
<div markdown="1">
{% highlight typescript %}
// importing base entities
import { ISPList, ISPView } from '../common/SPEntities';

/**
 * Data Helpers interface
 */
export interface IDataHelper {
  /**
   * API to get lists from the source
   */
  getLists(): Promise<ISPList[]>;
  /**
   * API to get views from the source
   */
  getViews(listId: string): Promise<ISPView[]>;
}
{% endhighlight %}
</div>
<br />Now let's create a mock helper. We'll use some static data for test. The mock helper will be created in <span style="font-family: &quot;courier new&quot; , &quot;courier&quot; , monospace;">DataHelperMock.ts</span><span style="font-family: inherit;">&nbsp;file.</span><br />
<div markdown="1">
{% highlight typescript %}
import {
  IWebPartContext
} from '@microsoft/sp-client-preview';
import { ISPList, ISPView } from '../common/SPEntities';
import { IDataHelper } from './DataHelperBase';

/**
 * MOCK data helper. Gets data from hardcoded values
 */
export class DataHelperMock implements IDataHelper {
  /**
   * hardcoded collection of lists
   */
  private static _lists: ISPList[] = [{ Title: 'Test 1', Id: '1' }, { Title: 'Test 2', Id: '2' }, { Title: 'Test 3', Id: '3' }];
  /**
   * hardcoded collection of views
   */
  private static _views: ISPView[] = [{ Title: 'All Items', Id: '1', ListId: '1' }, { Title: 'Demo', Id: '2', ListId: '1' }, { Title: 'All Items', Id: '1', ListId: '2' }, { Title: 'All Items', Id: '1', ListId: '3' }];

  /**
   * API to get lists from the source
   */
  public getLists(): Promise<ISPList[]> {
    return new Promise<ISPList[]>((resolve) => {
      resolve(DataHelperMock._lists);
    });
  }

  /**
   * API to get views from the source
   */
  public getViews(listId: string): Promise<ISPView[]> {
    return new Promise<ISPView[]>((resolve) => {
      const result: ISPView[] = DataHelperMock._views.filter((value, index, array) => {
        return value.ListId === listId;
      });
      resolve(result);
    });
  }
}

{% endhighlight %}
</div>
<span style="font-family: inherit;"><br /></span><span style="font-family: inherit;">Time to create SharePoint data helper. For this data helper we'll need a web part context and especially&nbsp;</span><span class="s1" style="font-family: &quot;courier new&quot; , &quot;courier&quot; , monospace;">pageContext</span><span class="s2" style="font-family: &quot;courier new&quot; , &quot;courier&quot; , monospace;">.</span><span class="s1" style="font-family: &quot;courier new&quot; , &quot;courier&quot; , monospace;">web</span><span class="s2" style="font-family: &quot;courier new&quot; , &quot;courier&quot; , monospace;">.</span><span class="s1"><span style="font-family: &quot;courier new&quot; , &quot;courier&quot; , monospace;">absoluteUrl</span><span style="font-family: inherit;">&nbsp;property that will be used to create a request url. Let's create a </span><span style="font-family: &quot;courier new&quot; , &quot;courier&quot; , monospace;">DataHelperSP.ts</span><span style="font-family: inherit;">&nbsp;file and implement the helper there:</span></span><br />
<div markdown="1">
{% highlight typescript %}
import {
  IWebPartContext
} from '@microsoft/sp-client-preview';
import { ISPList, ISPView, ISPLists, ISPViews } from '../common/SPEntities';
import { IDataHelper } from './DataHelperBase';

/**
 * List with views interface
 */
interface ISPListWithViews extends ISPList {
  /**
   * List Views
   */
  Views: ISPView[];
}

/**
 * SharePoint Data Helper class.
 * Gets information from current web
 */
export class DataHelperSP implements IDataHelper {
  /**
   * Web part context
   */
  public context: IWebPartContext;

  /**
   * Loaded lists
   */
  private _lists: ISPListWithViews[];

  /**
   * ctor
   */
  public constructor(_context: IWebPartContext) {
    this.context = _context;
  }

  /**
   * API to get lists from the source
   */
  public getLists(): Promise<ISPList[]> {
    return this.context.httpClient.get(this.context.pageContext.web.absoluteUrl + `/_api/web/lists?$filter=Hidden eq false`) // sending the request to SharePoint REST API
      .then((response: Response) => { // httpClient.get method returns a response object where json method creates a Promise of getting result
        return response.json(); 
      }).then((response: ISPLists) => { // response is an ISPLists object
        return response.value;
      });
  }

  /**
   * API to get views from the source
   */
  public getViews(listId: string): Promise<ISPView[]> {
    if (listId &amp;&amp; listId == '-1' || listId == '0')
      return new Promise<ISPView[]>((resolve) => {
      resolve(new Array<ISPView>());
    });

    //
    // trying to get views from cache
    //
    const lists: ISPListWithViews[] = this._lists &amp;&amp; this._lists.length &amp;&amp; this._lists.filter((value, index, array) => { return value.Id === listId; });

    if (lists &amp;&amp; lists.length) {
      return new Promise<ISPView[]>((resolve) => {
        resolve(lists[0].Views);
      });
    }
    else {
      return this.context.httpClient.get(this.context.pageContext.web.absoluteUrl + '/_api/web/lists(\'' + listId + '\')/views') // requesting views from SharePoint REST API
        .then((response: Response) => { // httpClient.get method returns a response object where json method creates a Promise of getting result
          return response.json();
        }).then((response: ISPViews) => { // response is an ISPView object
          var views = response.value;
          if (!this._lists || !this._lists.length)
            this._lists = new Array<ISPListWithViews>();
          this._lists.push({ Id: listId, Title: '', Views: views }); // saving views to cache
          return views;
        });
    }
  }
{% endhighlight %}
</div>
<span class="s1"><span style="font-family: inherit;"><br /></span></span><span class="s1"><span style="font-family: inherit;">The final part is to create a data helper factory that will create a data helper based on current environment. Let's do it in a </span><span style="font-family: &quot;courier new&quot; , &quot;courier&quot; , monospace;">DataHelpersFactory.ts</span><span style="font-family: inherit;">&nbsp;file.</span></span><br />
<div markdown="1">
{% highlight typescript %}
import {
  IWebPartContext // importing web part context class
} from '@microsoft/sp-client-preview';
import { EnvironmentType } from '@microsoft/sp-client-base'; // importing EnvironmentType enum to check current environment
//
// importing data helpers
//
import { IDataHelper } from './DataHelperBase';
import { DataHelperMock } from './DataHelperMock';
import { DataHelperSP } from './DataHelperSP';

/**
 * Factory object to create data helper based on current EnvironmentType
 */
export class DataHelpersFactory {
  /**
   * API to create data helper
   * @context: web part context
   */
  public static createDataHelper(context: IWebPartContext): IDataHelper {
    if (context.environment.type === EnvironmentType.Local) {
      return new DataHelperMock();
    }
    else {
      return new DataHelperSP(context);
    }
  }
}

{% endhighlight %}
</div>
<span class="s1"><span style="font-family: inherit;"><br /></span></span><span class="s1"><span style="font-family: inherit;">That's it. We have everything we need to get information from mock store as well as from SharePoint. It means that it can be used on your local server or on workbench.aspx from your dev site collection.</span></span><br /><span class="s1"><span style="font-family: inherit;">Let's test the results. In web part file add imports for factory, data helpers interface and base SP Entities:</span></span><br />
<div markdown="1">
{% highlight typescript %}
import { IDataHelper } from './data-helpers/DataHelperBase';
import { DataHelpersFactory } from './data-helpers/DataHelpersFactory';
import { ISPList, ISPView } from './common/SPEntities';

{% endhighlight %}
</div>
<span class="s1"><span style="font-family: inherit;"><br /></span></span><span class="s1"><span style="font-family: inherit;">Then, in </span><span style="font-family: &quot;courier new&quot; , &quot;courier&quot; , monospace;">render</span><span style="font-family: inherit;">&nbsp;method add code to retrieve lists and views and log information to console:</span></span><br />
<div markdown="1">
{% highlight typescript %}
const dataHelper: IDataHelper = DataHelpersFactory.createDataHelper(this.context);
dataHelper.getLists().then((lists: ISPList[]) => {
  console.log(lists); // logging lists
  if (lists.length) {
    const list: ISPList = lists[0];
    dataHelper.getViews(list.Id).then((views: ISPView[]) => {
      console.log(views); // logging views
    });
  }
});

{% endhighlight %}
</div>
<span class="s1"><span style="font-family: inherit;">And somewhere in console you'll find the results:</span></span><br /><div class="separator" style="clear: both; text-align: center;"><img border="0" src="{{site.baseurl}}/assets/images/posts/2016/2016-10-11-2.png" /></div><br /><br />
This is it for today. In the next post I'll describe how to create a custom field in a Property Pane that will use information from created data helpers and will dynamically update options in Views select when user selects a list.<span class="s1"><span style="font-family: inherit;"><br /></span></span><span class="s1"><span style="font-family: inherit;">Have fun!</span></span></div>