---
layout: post
title: Using PnP Field Controls in PnP ListView to Render Taxonomy, Pictures and Other
  Types of SharePoint Columns
date: '2020-03-10T08:20:00.002-07:00'
author: Alex Terentiev
tags:
- SharePoint Online
- Field Controls
- Microsoft 365
- SharePoint Framework
- Office 365
- SharePoint Development
- ListView
- PnP
- SharePoint
- PnP Reusable Controls
- SPFx
- Office Development
- SPFx Web Parts
modified_time: '2020-03-10T08:20:57.165-07:00'
featured_image_thumbnail: assets/images/posts/2020/web-part.png
featured_image: assets/images/posts/2020/web-part.png
blogger_id: tag:blogger.com,1999:blog-3066084330774405472.post-1077381746766294427
blogger_orig_url: http://blog.aterentiev.com/2020/03/using-pnp-field-co
---
This blog post is based on <a href="https://github.com/pnp/sp-dev-fx-webparts/tree/master/samples/react-pnp-controls-list-view-fields" target="_blank">SPFx web part sample</a> that shows how to use <a href="https://pnp.github.io/sp-dev-fx-controls-react/controls/fields/main/" target="_blank">PnP Field Controls</a> to render different types of SharePoint columns in <a href="https://pnp.github.io/sp-dev-fx-controls-react/controls/ListView/" target="_blank">ListView</a> control.<br /><a name='more'></a><h2>Objective</h2>Let's say we have a SharePoint list with such columns as Lookup, Choice, Hyperlink or Picture, etc. And we want to display items from this list in our SPFx web part.<br />In this blog post I will use the list as shown below:<br /><img border="0" src="{{site.baseurl}}/assets/images/posts/2020/sp-list.png" /><img border="0" src="{{site.baseurl}}/assets/images/posts/2020/list-cols.png" /><br />We want to display this list in our web part similarly to the OOTB rendering.<br /><h2>Getting the Items</h2>First thing to do - we need to request the list items from SharePoint.<br />There are several way how you can do that: <br /><ul><li>Using SharePoint REST API</li><li>Using Microsoft Graph</li><li>Using PnPJS library</li></ul>I will use PnPJS. And to be more precise: <span class="code">getItemsByCAMLQuery</span> method of <span class="code">IList</span> object. This method allows to correctly get labels for Managed Metadata columns (selected terms). And this is required for the selected list. So, let's install PnPJS module to our project: <br />
<div markdown="1">
{% highlight console %}
npm i --save @pnp/sp

{% endhighlight %}
</div>
Now let's import all needed instances in our web part's <span class="code">ts</span> file: <br />
<div markdown="1">
{% highlight typescript %}
import { sp } from '@pnp/sp';
import '@pnp/sp/webs';
import '@pnp/sp/lists';
import '@pnp/sp/items';

{% endhighlight %}
</div>
And let's request the items in web part's <span class="code">onInit</span> method: <br />
<div markdown="1">
{% highlight typescript %}
private _items: any[];
protected async onInit(): Promise&lt;void&gt; {
  await super.onInit();

  sp.setup({
    spfxContext: this.context
  });

  const countries = await sp.web.lists.getByTitle('Country').items.get(); // getting Country items to get lookup values

  this._items = await sp.web.lists.getByTitle('Journeys').getItemsByCAMLQuery({
    ViewXml: ''
  });

  // getting lookup values by ids
  this._items.forEach(item =&gt; {
    const visitedCountriesIds: number[] = item['VisitedCountriesId'] as number[];
    item['VisitedCountries'] = countries.filter(c =&gt; visitedCountriesIds.indexOf(c['ID']) !== -1).map(c =&gt; c['Title']);
  });

}

{% endhighlight %}
</div>
This request will return items in the format shown below:<br />
<div markdown="1">
{% highlight javascript %}
[
  {
    "Title": "Teen Vacation",
    "JourneyDate": "2000-07-10T00:00:00",
    "VisitedCountriesId": [
      5
    ],
    "Experience": "Good",
    "Picture": {
      "Description": "Hungray",
      "Url": "https://www.coe.int/documents/14577325/16404142/HUN_publication_report/6bac7580-94d7-1cf7-ba59-d3c74228fbad?t=1569512959000"
    },
    "JourneyType": {
      "Label": "Leisure",
      "TermGuid": "2262759a-0902-4ec6-87e0-910abb935f59",
      "WssId": 1
    },
    "VisitedCountries": [
      "Hungary"
    ]
  },
  {
    "Title": "Europe Journey",
    "JourneyDate": "2013-12-05T00:00:00",
    "VisitedCountriesId": [
      6,
      3
    ],
    "Experience": "Good",
    "Picture": {
      "Description": "greece",
      "Url": "https://static.coindesk.com/wp-content/uploads/2018/10/greece-1594689_1920-710x458.jpg?format=webp"
    },
    "JourneyType": {
      "Label": "Leisure",
      "TermGuid": "2262759a-0902-4ec6-87e0-910abb935f59",
      "WssId": 1
    },
    "VisitedCountries": [
      "Greece",
      "France"
    ]
  }
]

{% endhighlight %}
</div>
As you can see, such columns as <span class="code">VisitedCountriesId</span>, Picture and others are complex object with multiple properties. And we are additionally processing <span class="code">VisitedCountriesIds</span> to add <span class="code">VisitedCountries</span> property to contain array of lookup values. <br /><h2>PnP ListView</h2>PnP <span class="code">ListView</span> component allows you to render collection of items. It is based on Office UI Fabric React <span class="code">DetailsList</span> and allow to render items similarly to Office 365 experience.<br />You have to provide a list of view fields as <span class="code">IViewField[]</span> that describes what columns to render and list of items as <span class="code">any[]</span>. You can read documentation on these parameters <a href="https://pnp.github.io/sp-dev-fx-controls-react/controls/ListView/" target="_blank">here</a>.<br />The problem with this controls is that it flattens the passed items. And for our example, for <span class="code">VisitedCountries</span> it will create a separate property for each array member: <br />
<div markdown="1">
{% highlight javascript %}
{
  "VisitedCountries.0": "Greece",
  "VisitedCountries.1": "France"
}

{% endhighlight %}
</div>
Such a format doesn't allow to correctly specify <span class="code">IViewField</span> instance to render array.<br />Good thing is - we can define render function for each column and provide custom rendering for a cell there. <br /><h2>Field Controls</h2><a href="https://pnp.github.io/sp-dev-fx-controls-react/controls/fields/main/" target="_blank">PnP Field Controls</a> were initially created to be used in SPFx Field Customizers to provide rendering of the fields similar to out of the box experience.<br />But technically we can use them in web parts as well. And, as a result, we can use them to provide custom rendering for <span class="code">ListView</span> cells. <br /><h2>Combining Everything</h2>So, let's use Field Controls in our <span class="code">ListView</span>.<br />And first thing - add all needed imports: <br />
<div markdown="1">
{% highlight typescript %}
import { ListView, IViewField } from '@pnp/spfx-controls-react/lib/ListView';
import { FieldTextRenderer } from '@pnp/spfx-controls-react/lib/FieldTextRenderer';
import { FieldDateRenderer } from '@pnp/spfx-controls-react/lib/FieldDateRenderer';
import { FieldLookupRenderer } from '@pnp/spfx-controls-react/lib/FieldLookupRenderer';
import { FieldUrlRenderer } from '@pnp/spfx-controls-react/lib/FieldUrlRenderer';
import { FieldTaxonomyRenderer } from '@pnp/spfx-controls-react/lib/FieldTaxonomyRenderer';
import { ISPFieldLookupValue } from '@pnp/spfx-controls-react/lib/Common';

{% endhighlight %}
</div>
For Title and Experience (Choice) we will use simple <span class="code">FieldTextRenderer</span>: <br />
<div markdown="1">
{% highlight typescript %}
private _renderTitle(item?: any): any {
  return &lt;FieldTextRenderer
    text={item.Title}
  /&gt;;
}

private _renderExperience(item?: any): any {
  const experience = item['Expirience'];

  return &lt;FieldTextRenderer
    text={experience}
  /&gt;;
}

{% endhighlight %}
</div>
For date - <span class="code">FieldDateRenderer</span>: <br />
<div markdown="1">
{% highlight typescript %}
private _renderDate(item?: any): any {
  const date = new Date(item['JourneyDate']);
  return &lt;FieldDateRenderer
    text={date.toLocaleDateString()} /&gt;;
}

{% endhighlight %}
</div>
For the Picture (Hyperlink or Picture) we need to get 2 properties of the flattened item: <span class="code">Picture.Url</span> and <span class="code">Picture.Description</span> to use <span class="code">FieldUrlRenderer</span><br />
<div markdown="1">
{% highlight typescript %}
private _renderPicture(item?: any): any {

  return &lt;FieldUrlRenderer
    url={item['Picture.Url']}
    isImageUrl={true}
    className={styles.image}
    text={item['Picture.Description'] || ''} /&gt;;
}

{% endhighlight %}
</div>
Similarly, for JorneyType (Managed Metadata) we need <span class="" code="">JourneyType.Label</span> and <span class="code">JourneyType.TermGuid</span> to use <span class="code">FieldTaxonomyRenderer</span>: <br />
<div markdown="1">
{% highlight typescript %}
private _renderJourneyType(item?: any) {

  return &lt;FieldTaxonomyRenderer
    terms={[{
      Label: item['JourneyType.Label'],
      TermID: item['JourneyType.TermGuid']
    }]} /&gt;;
  }

{% endhighlight %}
</div>
The most interesting part is VisitedCoutries. As this is a lookup column we want to displayed referenced item when user clicks on the lookup value. We can display this item either in a separate tab or in a popup. I will show the second option. So, first let's use <span class="code">FieldLookupRenderer</span> to render the values and handle <span class="code">onClick</span>: <br />
<div markdown="1">
{% highlight typescript %}
private _renderCountries(item?: any, index?: number): any {
  //
  // ListView item contains "flattened" information
  // So, we're getting original item first
  //

  const originalItem = this.props.items[index!];
  const visitedCountriesIds = originalItem['VisitedCountriesId'];
  const visitedCountries = originalItem['VisitedCountries'];

  const lookups: ISPFieldLookupValue[] = visitedCountries.map((vc, idx) =&gt; {
    return {
      lookupId: visitedCountriesIds[idx].toString(),
      lookupValue: vc
    };
  });

    return &lt;FieldLookupRenderer
      lookups={lookups}
      onClick={args =&gt; {
        this.setState({
          selectedLookupId: args.lookup.lookupId
        });
      }} /&gt;;
  }

{% endhighlight %}
</div>
And if <span class="code">selectedLookupId</span> is set, we'll use one more PnP Reusable Control <span class="code">IFrameDialog</span> to render list item display form as a dialog: <br />
<div markdown="1">
{% highlight typescript %}
import { IFrameDialog } from '@pnp/spfx-controls-react/lib/IFrameDialog';
import { DialogType } from 'office-ui-fabric-react/lib/Dialog';
//...
public render(): React.ReactElement&lt;IPnPListViewProps&gt; {
//...
{this.state.selectedLookupId &amp;&amp;
  &lt;IFrameDialog
    hidden={false}
    url={`${this.props.context.pageContext.web.absoluteUrl}${this._lookupFieldDispWebRelativeUrl.replace('{0}', this.state.selectedLookupId.toString())}`}
    iframeOnLoad={iframe =&gt; {
      const iframeWindow: Window = iframe.contentWindow;
      const iframeDocument: Document = iframeWindow.document;

      const s4Workspace: HTMLDivElement = iframeDocument.getElementById('s4-workspace') as HTMLDivElement;
      s4Workspace.style.height = iframe.style.height;

      s4Workspace.scrollIntoView();
    }}
    onDismiss={() =&gt; {
      this.setState({
        selectedLookupId: undefined
      });
    }}
    modalProps={{ "{{" }}
      isBlocking: true
    }}
    dialogContentProps={{ "{{" }}
      type: DialogType.close,
      showCloseButton: true
    }}
    width={'570px'}
    height={'250px'} /&gt;}
//...
}

{% endhighlight %}
</div>
The constructed <span class="code">url</span> parameter will look like that: <span class="code">https://tenant.sharepoint.com/sites/your-site/Lists/Journeys/DispForm.aspx?ID=item-id&amp;IsDlg=1</span><br />The final code of the component: <br />
<div markdown="1">
{% highlight typescript %}
export default class PnPListView extends React.Component&lt;IPnPListViewProps, IPnPListViewState&gt; {

  private readonly _countriesLookupFieldId = '5e037cea-aaef-40dd-895f-90442114016f';
  private readonly _lookupFieldDispWebRelativeUrl = '/Lists/Country/DispForm.aspx?ID={0}&amp;RootFolder=*&amp;IsDlg=1';

  private readonly _fields: IViewField[] = [{
    name: 'Title',
    displayName: 'Title',
    minWidth: 150,
    maxWidth: 250,
    render: this._renderTitle
  }, {
    name: 'JourneyDate',
    displayName: 'Journey Date',
    render: this._renderDate
  }, {
    name: 'VisitedCountries',
    displayName: 'Visited Countries',
    minWidth: 100,
    render: (item, index) =&gt; { return this._renderCountries(item, index); }
  }, {
    name: 'Experience',
    displayName: 'Experience',
    minWidth: 100,
    render: this._renderExperience
  }, {
    name: 'Picture',
    displayName: 'Picture',
    minWidth: 150,
    render: this._renderPicture
  }, {
    name: 'JourneyType',
    displayName: 'Journey Type',
    minWidth: 100,
    render: this._renderJourneyType
  }];

  constructor(props: IPnPListViewProps) {
    super(props);

    this.state = {};
  }


  public render(): React.ReactElement&lt;IPnPListViewProps&gt; {
    return (
      &lt;div className={styles.pnPListView}&gt;
        &lt;ListView items={this.props.items} viewFields={this._fields} /&gt;
        {this.state.selectedLookupId &amp;&amp;
          &lt;IFrameDialog
            hidden={false}
            url={`${this.props.context.pageContext.web.absoluteUrl}${this._lookupFieldDispWebRelativeUrl.replace('{0}', this.state.selectedLookupId.toString())}`}
            iframeOnLoad={iframe =&gt; {
              const iframeWindow: Window = iframe.contentWindow;
              const iframeDocument: Document = iframeWindow.document;

              const s4Workspace: HTMLDivElement = iframeDocument.getElementById('s4-workspace') as HTMLDivElement;
              s4Workspace.style.height = iframe.style.height;

              s4Workspace.scrollIntoView();
            }}
            onDismiss={() =&gt; {
              this.setState({
                selectedLookupId: undefined
              });
            }}
            modalProps={{ "{{" }}
              isBlocking: true
            }}
            dialogContentProps={{ "{{" }}
              type: DialogType.close,
              showCloseButton: true
            }}
            width={'570px'}
            height={'250px'} /&gt;}
      &lt;/div&gt;
    );
  }

  /**
   * Title column renderer
   * @param item ListView item
   */
  private _renderTitle(item?: any): any {
    return &lt;FieldTextRenderer
      text={item.Title}
    /&gt;;
  }

  /**
   * Date column renderer
   * @param item ListView item
   */
  private _renderDate(item?: any): any {
    const date = new Date(item['JourneyDate']);
    return &lt;FieldDateRenderer
      text={date.toLocaleDateString()} /&gt;;
  }

  /**
   * Countries (Multi Lookup) column renderer
   * @param item ListView item
   * @param index item index
   */
  private _renderCountries(item?: any, index?: number): any {
    //
    // ListView item contains "flattened" information
    // So, we're getting original item first
    //

    const originalItem = this.props.items[index!];
    const visitedCountriesIds = originalItem['VisitedCountriesId'];
    const visitedCountries = originalItem['VisitedCountries'];

    const lookups: ISPFieldLookupValue[] = visitedCountries.map((vc, idx) =&gt; {
      return {
        lookupId: visitedCountriesIds[idx].toString(),
        lookupValue: vc
      };
    });

    return &lt;FieldLookupRenderer
      lookups={lookups}
      onClick={args =&gt; {
        this.setState({
          selectedLookupId: args.lookup.lookupId
        });
      }} /&gt;;
  }

  /**
   * Experience (Choice) column renderer
   * @param item ListView item
   */
  private _renderExperience(item?: any): any {
    const experience: string = item['Expirience'];

    return &lt;FieldTextRenderer
      text={experience}
    /&gt;;
  }

  /**
   * Picture (Hyperlink or Picture) column renderer
   * @param item ListView item
   */
  private _renderPicture(item?: any): any {

    return &lt;FieldUrlRenderer
      url={item['Picture.Url']}
      isImageUrl={true}
      className={styles.image}
      text={item['Picture.Description'] || ''} /&gt;;
  }

  /**
   * JourneyType (Managed Metadata) column renderer
   * @param item ListView item
   */
  private _renderJourneyType(item?: any) {

    return &lt;FieldTaxonomyRenderer
      terms={[{
        Label: item['JourneyType.Label'],
        TermID: item['JourneyType.TermGuid']
      }]} /&gt;;
  }
}

{% endhighlight %}
</div>
And this is how our list will look:<br /><img border="0" src="{{site.baseurl}}/assets/images/posts/2020/web-part.png" /><br />And lookup popup:<br /><img border="0" src="{{site.baseurl}}/assets/images/posts/2020/popup.png" /><br /><h2>Conclusion</h2>As you can see it's pretty easy to combine different PnP Reusable Controls to work together. Using the same technique you can render other SharePoint column types like People or Group, multiline text, and others.<br />As mentioned in the beginning of the post, you can find all the code <a href="https://github.com/SharePoint/sp-dev-fx-webparts/tree/master/samples/react-pnp-controls-list-view-fields" target="_blank">here</a><br /><br /><br />That's all for today!<br />Have fun! for today!<br />Have fun!