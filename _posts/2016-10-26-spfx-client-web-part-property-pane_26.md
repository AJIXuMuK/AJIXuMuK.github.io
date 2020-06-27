---
layout: post
title: 'SPFx. Client Web Part Property Pane Dependent Properties. Part II: Dependent
  Properties Control.'
date: '2016-10-26T15:54:00.000-07:00'
author: Alex Terentiev
tags:
- JavaScript
- SharePoint Online
- SPFx
- Client Web Part
- SharePoint Framework
- Future of SharePoint
- SharePoint
modified_time: '2016-10-28T16:04:15.217-07:00'
thumbnail: https://3.bp.blogspot.com/-rlde-CtbnBM/WBOnktZTdbI/AAAAAAAAAa8/8SkI-6Grj4gnW0i3l8s7_ncUcLNFueRdACLcB/s72-c/Screen%2BShot%2B2016-10-28%2Bat%2B12.31.15%2BPM.png
blogger_id: tag:blogger.com,1999:blog-3066084330774405472.post-5681112166172475384
blogger_orig_url: http://blog.aterentiev.com/2016/10/spfx-client-web-part-property-pane_26.html
---

This is the second post of three that explain how to add dependent properties to Client Web Part Property Pane.<br />As an example of such properties I decided to use Lists dropdown and Views dropdown. Views dropdown is populated based on selected List.<br />Here is a content of 3 posts: As a result of these three posts we'll have a fully working web part with ability to select List View in Property Pane:<br /><div class="separator" style="clear: both; text-align: center;"><a href="https://3.bp.blogspot.com/-rlde-CtbnBM/WBOnktZTdbI/AAAAAAAAAa8/8SkI-6Grj4gnW0i3l8s7_ncUcLNFueRdACLcB/s1600/Screen%2BShot%2B2016-10-28%2Bat%2B12.31.15%2BPM.png" imageanchor="1" style="margin-left: 1em; margin-right: 1em;"><img border="0" height="312" src="https://3.bp.blogspot.com/-rlde-CtbnBM/WBOnktZTdbI/AAAAAAAAAa8/8SkI-6Grj4gnW0i3l8s7_ncUcLNFueRdACLcB/s640/Screen%2BShot%2B2016-10-28%2Bat%2B12.31.15%2BPM.png" width="640" /></a></div><br /><ol><li><a href="http://tricky-sharepoint.blogspot.com/2016/10/spfx-client-web-part-property-pane.html" target="_blank">SPFx. Client Web Part Property Pane Dependent Properties. Part I: Preparation.</a></li><li>SPFx. Client Web Part Property Pane Dependent Properties. Part II: Dependent Properties Control. (current post)</li><li><a href="http://tricky-sharepoint.blogspot.com/2016/10/spfx-client-web-part-property-pane_27.html" target="_blank">SPFx. Client Web Part Property Pane Dependent Properties. Part III: Styling Dependent Properties Control as Office UI Fabric dropdowns. Knockout version.</a></li></ol><div>All the code is available on GitHub (<a href="https://github.com/AJIXuMuK/SPFx/tree/master/dep-props-custom-class" target="_blank">https://github.com/AJIXuMuK/SPFx/tree/master/dep-props-custom-class</a>)<br />I'm using Visual Studio Code IDE for the tutorial.</div><div style="background-color: white; color: #666666; font-size: 13.2px;"><a name='more'></a><span style="font-family: &quot;trebuchet ms&quot; , &quot;trebuchet&quot; , &quot;verdana&quot; , sans-serif;"><br /></span><span style="font-family: &quot;trebuchet ms&quot; , &quot;trebuchet&quot; , &quot;verdana&quot; , sans-serif;">This post describes how to create custom Property Pane Field with 2 simple unstyled dropdowns (</span><span style="font-family: &quot;courier new&quot; , &quot;courier&quot; , monospace;">&lt;select&gt;</span><span style="font-family: &quot;trebuchet ms&quot; , &quot;trebuchet&quot; , &quot;verdana&quot; , sans-serif;">&nbsp;elements).</span><br /><span style="font-family: &quot;trebuchet ms&quot; , &quot;trebuchet&quot; , &quot;verdana&quot; , sans-serif;"><br /></span><br /><div style="font-family: &quot;trebuchet ms&quot;, trebuchet, verdana, sans-serif;">The main idea of creating this web part is to be able to populate Views dropdown dynamically when user selects value in Lists dropdown.</div></div><div style="background-color: white; color: #666666; font-size: 13.2px;"><span style="font-family: &quot;trebuchet ms&quot; , &quot;trebuchet&quot; , &quot;verdana&quot; , sans-serif;">First idea was to use 2 </span><span style="font-family: &quot;courier new&quot; , &quot;courier&quot; , monospace;">PropertyPaneDropdown</span><span style="font-family: &quot;trebuchet ms&quot; , &quot;trebuchet&quot; , &quot;verdana&quot; , sans-serif;">&nbsp;controls. If user selects something from one dropdown we're doing something to get data and push this data to the second dropdown.</span></div><div style="background-color: white; color: #666666; font-size: 13.2px;"><span style="font-family: &quot;trebuchet ms&quot; , &quot;trebuchet&quot; , &quot;verdana&quot; , sans-serif;">It won't work for at least 3 reasons:</span></div><div style="background-color: white;"><ol><li><span style="color: #666666; font-family: &quot;trebuchet ms&quot; , &quot;trebuchet&quot; , &quot;verdana&quot; , sans-serif;"><span style="font-size: 13.2px;">Data request is an asynchronous operation. But I didn't find any method to override that will wait for this data before rerender the property pane.</span></span></li><li><span style="color: #666666;"><span style="font-family: &quot;trebuchet ms&quot; , &quot;trebuchet&quot; , &quot;verdana&quot; , sans-serif;"><span style="font-size: 13.2px;">Property Pane rerender and&nbsp;</span></span><span style="font-family: &quot;courier new&quot; , &quot;courier&quot; , monospace;"><span style="font-size: 13.2px;">propertyPaneSettings</span></span><span style="font-family: &quot;trebuchet ms&quot; , &quot;trebuchet&quot; , &quot;verdana&quot; , sans-serif;"><span style="font-size: 13.2px;">&nbsp;property getter are called dynamically only if web part property pane behavior is set to Reactive. It means that you won't be able to do anything in non-Reactive property pane (when there is an Apply button at the bottom of the pane).</span></span></span></li><li><span style="color: #666666;"><span style="font-family: &quot;trebuchet ms&quot; , &quot;trebuchet&quot; , &quot;verdana&quot; , sans-serif;"><span style="font-size: 13.2px;">You can't initiate Property Pane refresh from your custom field or from web part.</span></span></span></li></ol></div><div><span style="color: #666666; font-family: &quot;trebuchet ms&quot; , &quot;trebuchet&quot; , &quot;verdana&quot; , sans-serif;"><span style="font-size: 13.2px;">That's why the only way to implement dependent properties is to create custom Property Pane Field <b>that will contain both dropdowns in it.</b></span></span></div><div><span style="color: #666666; font-family: &quot;trebuchet ms&quot; , &quot;trebuchet&quot; , &quot;verdana&quot; , sans-serif;"><span style="font-size: 13.2px;"><b><br /></b></span></span></div><div><span style="color: #666666;"><span style="font-size: 13.2px;"><span style="font-family: &quot;trebuchet ms&quot; , &quot;trebuchet&quot; , &quot;verdana&quot; , sans-serif;">Let's consider that our custom Property Pane Field is named </span><span style="font-family: &quot;courier new&quot; , &quot;courier&quot; , monospace;">PropertyPaneViewSelector</span><span style="font-family: inherit;">. </span><span style="font-family: &quot;trebuchet ms&quot; , sans-serif;">It will have 2 properties that should be available in web part (and saved with web part): List Id and View Id.</span></span></span></div><div><span style="color: #666666;"><span style="font-size: 13.2px;"><span style="font-family: &quot;trebuchet ms&quot; , sans-serif;">To make these properties available in web part we need to update Web Part Properties interface. In our case it is named as&nbsp;</span></span></span><span style="color: #666666;"><span style="font-size: 13.2px;"><span style="font-family: &quot;courier new&quot; , &quot;courier&quot; , monospace;">IDepPropsWebPartProps</span><span style="font-family: &quot;trebuchet ms&quot; , sans-serif;">. We can either add 2 separate properties there or create a complex object with 2 properties in it. I think that the second option is logically better because List Id and View Id will be populated from a single Property Pane Field.</span></span></span></div><div><span style="color: #666666; font-family: &quot;trebuchet ms&quot; , sans-serif; font-size: 13.2px;">So let's create an interface for the complex property object first (I put it in /webparts/depProps/controls/Common.ts) and then change the </span><span style="color: #666666; font-family: &quot;courier new&quot; , &quot;courier&quot; , monospace; font-size: 13.2px;">IDepPropsWebPartProps</span></div><div><span style="color: #666666; font-family: &quot;courier new&quot; , &quot;courier&quot; , monospace; font-size: 13.2px;">IPropertyPaneViewSelectorProps</span></div>
<div markdown="1">
{% highlight typescript %}
/**
 * Complex object to define property in a web part
 */
export interface IPropertyPaneViewSelectorProps {
  /**
   * Selected list id
   */
  listId: string;
  /**
   * Selected view id
   */
  viewId: string;
}

{% endhighlight %}
</div>
<div><span style="color: #666666; font-family: &quot;trebuchet ms&quot; , sans-serif; font-size: 13.2px;">And in&nbsp;</span><span style="font-family: &quot;courier new&quot; , &quot;courier&quot; , monospace;">IDepPropsWebPartProps.ts</span></div>
<div markdown="1">
{% highlight typescript %}
import { IPropertyPaneViewSelectorProps } from './controls/Common';

export interface IDepPropsWebPartProps {
  depProps: IPropertyPaneViewSelectorProps;
}

{% endhighlight %}
</div>
<div><span style="color: #666666; font-family: &quot;trebuchet ms&quot; , sans-serif; font-size: 13.2px;">Now we can access these props in web part class like </span><span style="font-family: &quot;courier new&quot; , &quot;courier&quot; , monospace;">this.properties.depProps.listId and this.properties.depProps.viewId.&nbsp;</span><br /><span style="color: #666666; font-family: &quot;trebuchet ms&quot; , sans-serif; font-size: 13.2px;">Let's set defaults for these properties. For that we need to go to&nbsp;</span><span style="font-family: &quot;courier new&quot; , &quot;courier&quot; , monospace;">DepPropsWebPart.manifest.json</span>&nbsp;<span style="color: #666666; font-family: &quot;trebuchet ms&quot; , sans-serif; font-size: 13.2px;">and in </span><span style="font-family: &quot;courier new&quot; , &quot;courier&quot; , monospace;">preconfiguredEntries</span><span style="color: #666666; font-family: &quot;trebuchet ms&quot; , sans-serif; font-size: 13.2px;">&nbsp;object change </span><span style="font-family: &quot;courier new&quot; , &quot;courier&quot; , monospace;">properties</span><span style="color: #666666; font-family: &quot;trebuchet ms&quot; , sans-serif; font-size: 13.2px;">&nbsp;property like that:</span><br />
<div markdown="1">
{% highlight typescript %}
"preconfiguredEntries": [{
    //
    // ... some other properties 
    //
    "properties": {
      "depProps": {
        "listId": "-1",
        "viewId": "-1"
      }
    }
  }]

{% endhighlight %}
</div>
<span style="color: #666666; font-family: &quot;trebuchet ms&quot; , sans-serif; font-size: 13.2px;">Let's change web part </span><span style="font-family: &quot;courier new&quot; , &quot;courier&quot; , monospace;">render</span><span style="color: #666666; font-family: &quot;trebuchet ms&quot; , sans-serif; font-size: 13.2px;">&nbsp;method to display currently selected List Id and View Id:</span><br />
<div markdown="1">
{% highlight typescript %}
public render(): void {
    this.domElement.innerHTML = `
      <div class="${styles.depProps}">
        <div class="${styles.container}">
          <div class="ms-Grid-row ms-bgColor-themeDark ms-fontColor-white ${styles.row}">
            <div class="ms-Grid-col ms-u-lg10 ms-u-xl8 ms-u-xlPush2 ms-u-lgPush1">
              <p class="ms-font-l ms-fontColor-white">Selected List Id: ${this.properties.depProps.listId}</p>
              <p class="ms-font-l ms-fontColor-white">Selected View Id: ${this.properties.depProps.viewId}</p>
            </div>
          </div>
        </div>
      </div>`;
  }

{% endhighlight %}
</div>
<span style="color: #666666; font-family: &quot;trebuchet ms&quot; , sans-serif; font-size: 13.2px;"><br /></span><span style="font-family: &quot;trebuchet ms&quot; , sans-serif;">Now let's start with custom Property Pane Field. To create such field we need to <b>create a class that implements&nbsp;</b></span><span style="font-family: &quot;courier new&quot; , &quot;courier&quot; , monospace;"><b>IPropertyPaneField</b></span><span style="color: #666666; font-family: &quot;trebuchet ms&quot; , sans-serif; font-size: 13.2px;"><b>&nbsp;interface:</b>.&nbsp;</span><br />
<div markdown="1">
{% highlight typescript %}
/**
 * PropertyPane field.
 */
export interface IPropertyPaneField<TProperties> {
    /**
     * Type of the PropertyPane field.
     */
    type: IPropertyPaneFieldType;
    /**
     * Target property from the web part's property bag.
     */
    targetProperty: string;
    /**
     * Strongly typed properties object. Specific to each field type.
     * Example: Checkbox has ICheckboxProps, TextField has ITextField props.
     *
      * @internalremarks - These props are from the office-ui-fabric-react.
      *   These props may not be extensive as the fabric-react ones. This is intentional.
      *     - We are not including any callbacks as part of the props, as this might end up breaking
      *       the internal flow and cause unwanted problems.
      *     - Currently discussions are going on whether to include styling elements or not. Based on
      *       the output of the discussions, changes to the styling related props will take place.
      *
      *   We are including only those which are supported by the web part framework.
     */
    properties: TProperties;
}

{% endhighlight %}
</div>
<span style="color: #666666; font-family: &quot;trebuchet ms&quot; , sans-serif; font-size: 13.2px;"><br /></span><span style="color: #666666; font-family: &quot;trebuchet ms&quot; , sans-serif; font-size: 13.2px;">As you can see it's a so called Generic interface and when implemented </span><span style="font-family: &quot;courier new&quot; , &quot;courier&quot; , monospace;">TProperties</span><span style="color: #666666; font-family: &quot;trebuchet ms&quot; , sans-serif; font-size: 13.2px;">&nbsp;should be changed to some specific interface. And one more thing about this interface - it should extend </span><span style="font-family: &quot;courier new&quot; , &quot;courier&quot; , monospace;">IPropertyPaneCustomFieldProps</span><span style="color: #666666; font-family: &quot;trebuchet ms&quot; , sans-serif; font-size: 13.2px;">&nbsp;if we want to be able to customize rendering. And this interface will be used to pass parameters to the constructor of our field. It means it should contain all the properties we need to get from web part to initialize the field correctly.</span><br /><span style="color: #666666; font-family: &quot;trebuchet ms&quot; , sans-serif; font-size: 13.2px;">So we need to think what properties we need in our field.</span><br /><br /><ul><li><span style="font-family: &quot;courier new&quot; , &quot;courier&quot; , monospace;">onRender</span><span style="color: #666666; font-family: &quot;trebuchet ms&quot; , sans-serif; font-size: 13.2px;">&nbsp;and </span><span style="font-family: &quot;courier new&quot; , &quot;courier&quot; , monospace;">onDispose?</span><span style="color: #666666; font-family: &quot;trebuchet ms&quot; , sans-serif; font-size: 13.2px;">&nbsp;from base interface</span></li><li><span style="font-family: &quot;courier new&quot; , &quot;courier&quot; , monospace;">listId</span><span style="color: #666666; font-family: &quot;trebuchet ms&quot; , sans-serif; font-size: 13.2px;">&nbsp;and </span><span style="font-family: &quot;courier new&quot; , &quot;courier&quot; , monospace;">viewId</span><span style="color: #666666; font-family: &quot;trebuchet ms&quot; , sans-serif; font-size: 13.2px;">&nbsp;- our main data</span></li><li><span style="color: #666666; font-family: &quot;trebuchet ms&quot; , sans-serif; font-size: 13.2px;">also we need some labels to display near dropdowns. Of course we can hardcode them or get from some source inside the field. But we can set them from web part. It will make our field more isolated and flexible. Let's name these properties as </span><span style="font-family: &quot;courier new&quot; , &quot;courier&quot; , monospace;">listLabel</span><span style="color: #666666; font-family: &quot;trebuchet ms&quot; , sans-serif; font-size: 13.2px;">&nbsp;and </span><span style="font-family: &quot;courier new&quot; , &quot;courier&quot; , monospace;">viewLabel</span><span style="color: #666666; font-family: &quot;trebuchet ms&quot; , sans-serif; font-size: 13.2px;">&nbsp;</span></li><li><span style="color: #666666; font-family: &quot;trebuchet ms&quot; , sans-serif; font-size: 13.2px;">we will use data helpers created in&nbsp;<a href="http://tricky-sharepoint.blogspot.com/2016/10/spfx-client-web-part-property-pane.html" target="_blank">previous</a>&nbsp;post. And these helpers use web part context. So we need this context in our properties as well.</span></li><li><span style="color: #666666; font-family: &quot;trebuchet ms&quot; , sans-serif; font-size: 13.2px;">we need to notify web part that something has changed in our field. For this purpose we'll add </span><span style="font-family: &quot;courier new&quot; , &quot;courier&quot; , monospace;">onPropertyChange</span><span style="color: #666666; font-family: &quot;trebuchet ms&quot; , sans-serif; font-size: 13.2px;">&nbsp;property in the interface to get the handler from a web part.</span></li></ul><div><span style="color: #666666; font-family: &quot;trebuchet ms&quot; , sans-serif; font-size: 13.2px;">Final interface will look like that (</span><span style="color: #666666; font-family: &quot;trebuchet ms&quot; , sans-serif; font-size: 13.2px;">/webparts/depProps/controls/Common.ts</span><span style="color: #666666; font-family: &quot;trebuchet ms&quot; , sans-serif; font-size: 13.2px;">):</span></div>
<div markdown="1">
{% highlight typescript %}
/**
 * PropertyPaneViewSelector component internal props
 */
export interface IPropertyPaneViewSelectorFieldPropsInternal extends IPropertyPaneCustomFieldProps {
  /**
   * Path to target property in web part properties
   */
  targetProperty: string;
  /**
   * web part context
   */
  context: IWebPartContext;
  /**
   * selected list id
   */
  listId: string;
  /**
   * selected view id
   */
  viewId: string;
  /**
   * onPropertyChange event handler
   */
  onPropertyChange(propertyPath: string, newValue: any): void;
  /**
   * Label to show in list dropdown
   */
  listLabel: string;
  /**
   * Label to show in view dropdown
   */
  viewLabel: string;
}

{% endhighlight %}
</div>
<div><span style="font-family: &quot;trebuchet ms&quot; , sans-serif; font-size: 13.2px;">You can ask why it is named with </span><span style="font-family: &quot;courier new&quot; , &quot;courier&quot; , monospace; font-size: 13.2px;">Internal</span><span style="font-family: &quot;trebuchet ms&quot; , sans-serif; font-size: 13.2px;">&nbsp;prefix. I'll describe it in a second.</span><br /><span style="font-family: &quot;trebuchet ms&quot; , sans-serif; font-size: 13.2px;">For now our custom class will look like that (/webparts/depProps/controls/PropertyPaneViewSelector.ts):</span><br />
<div markdown="1">
{% highlight typescript %}
/**
 * PropertyPaneViewSelector component
 */
class PropertyPaneViewSelector implements IPropertyPaneField<IPropertyPaneViewSelectorFieldPropsInternal> {
  /**
   * This is a Custom field
   */
  public type: IPropertyPaneFieldType = IPropertyPaneFieldType.Custom;
  /**
   * Path to target property in web part properties
   */
  public targetProperty: string;
  /**
   * component properties
   */
  public properties: IPropertyPaneViewSelectorFieldPropsInternal;

  /**
   * selected list id
   */
  public listId: string;
  /**
   * selected view id
   */
  public viewId: string;

  /**
   * ctor
   */
  public constructor(_targetProperty: string, _properties: IPropertyPaneViewSelectorFieldPropsInternal) {
    this.targetProperty = _targetProperty;
    this.properties = _properties;
    this.properties.onRender = this._render.bind(this); // applying custom rendering method
    this.properties.onDispose = this._dispose.bind(this); // applying custom disposing method
    this.listId = _properties.listId;
    this.viewId = _properties.viewId;
  }

  /**
   * Rendering the component
   */
  private _render(elem: HTMLElement): void {
    // custom rendering
  }

  private _dispose(elem: HTMLElement): void {
    // custom dispose
  }
}

{% endhighlight %}
</div>
<span style="background-color: white; color: #666666; font-family: &quot;trebuchet ms&quot; , &quot;trebuchet&quot; , &quot;verdana&quot; , sans-serif; font-size: 13.2px;"><br /></span></div><div><span style="background-color: white; color: #666666; font-family: &quot;trebuchet ms&quot; , &quot;trebuchet&quot; , &quot;verdana&quot; , sans-serif; font-size: 13.2px;">If you look at out-of-the-box Property Pane Fields, you'll see that field classes are not created directly. There is always a helper function that creates an instance of Field's class.</span><br /><span style="background-color: white; color: #666666; font-family: &quot;trebuchet ms&quot; , &quot;trebuchet&quot; , &quot;verdana&quot; , sans-serif; font-size: 13.2px;">Let's create helper function for our class. Also this function will allow us to hide </span><span style="background-color: white; color: #666666; font-family: &quot;courier new&quot; , &quot;courier&quot; , monospace; font-size: 13.2px;">onRender</span><span style="background-color: white; color: #666666; font-family: &quot;trebuchet ms&quot; , &quot;trebuchet&quot; , &quot;verdana&quot; , sans-serif; font-size: 13.2px;">&nbsp;and </span><span style="background-color: white; color: #666666; font-family: &quot;courier new&quot; , &quot;courier&quot; , monospace; font-size: 13.2px;">onDispose </span><span style="background-color: white; color: #666666; font-family: &quot;trebuchet ms&quot; , &quot;trebuchet&quot; , &quot;verdana&quot; , sans-serif; font-size: 13.2px;">as we want to implement them internally. That's why we created </span><span style="background-color: white; color: #666666; font-family: &quot;courier new&quot; , &quot;courier&quot; , monospace; font-size: 13.2px;">Internal</span><span style="background-color: white; color: #666666; font-family: &quot;trebuchet ms&quot; , &quot;trebuchet&quot; , &quot;verdana&quot; , sans-serif; font-size: 13.2px;">&nbsp;properties interface. Let's create 'external' one (/webparts/depProps/controls/Common.ts):</span><br />
<div markdown="1">
{% highlight typescript %}
/**
 * PropertyPaneViewSelector component public props
 */
export interface IPropertyPaneViewSelectorFieldProps {
  /**
   * web part context
   */
  context: IWebPartContext;
  /**
   * selected list id
   */
  listId: string;
  /**
   * selected view id
   */
  viewId: string;
  /**
   * onPropertyChange event handler
   */
  onPropertyChange(propertyPath: string, newValue: any): void;
  /**
   * Label to show in list dropdown
   */
  listLabel: string;
  /**
   * Label to show in view dropdown
   */
  viewLabel: string;
}
{% endhighlight %}
</div>
<span style="background-color: white; color: #666666; font-family: &quot;trebuchet ms&quot; , &quot;trebuchet&quot; , &quot;verdana&quot; , sans-serif; font-size: 13.2px;">and a helper function (/webparts/depProps/controls/PropertyPaneViewSelector.ts):</span><br />
<div markdown="1">
{% highlight typescript %}
/**
 * Helper method to create a ViewSelectorField on the PropertyPane.
 * @param targetProperty - Target property the viewselector is associated to.
 * @param properties - Strongly typed viewselector properties.
 */
export function PropertyPaneViewSelectorField(targetProperty: string, properties: IPropertyPaneViewSelectorFieldProps): IPropertyPaneField<IPropertyPaneViewSelectorFieldProps> {
  var internalProps: IPropertyPaneViewSelectorFieldPropsInternal = {
    context: properties.context,
    listId: properties.listId,
    viewId: properties.viewId,
    listLabel: properties.listLabel,
    viewLabel: properties.viewLabel,
    onPropertyChange: properties.onPropertyChange,
    targetProperty: targetProperty,
    onRender: null,
    onDispose: null
  };
  return new PropertyPaneViewSelector(targetProperty, internalProps);
}

{% endhighlight %}
</div>
<span style="background-color: white; color: #666666; font-family: &quot;trebuchet ms&quot; , &quot;trebuchet&quot; , &quot;verdana&quot; , sans-serif; font-size: 13.2px;">Now we can call the helper function from web part </span><span style="background-color: white; color: #666666; font-family: &quot;courier new&quot; , &quot;courier&quot; , monospace; font-size: 13.2px;">propertyPaneSettings</span><span style="background-color: white; color: #666666; font-family: &quot;trebuchet ms&quot; , &quot;trebuchet&quot; , &quot;verdana&quot; , sans-serif; font-size: 13.2px;">&nbsp;getter:</span><br />
<div markdown="1">
{% highlight typescript %}
protected get propertyPaneSettings(): IPropertyPaneSettings {
  return {
    pages: [
      {
        header: {
          description: strings.PropertyPaneDescription
        },
        groups: [
          {
            groupName: strings.BasicGroupName,
            groupFields: [
              PropertyPaneViewSelectorField('depProps', {
                context: this.context,
                listId: this.properties.depProps &amp;&amp; this.properties.depProps.listId,
                viewId: this.properties.depProps &amp;&amp; this.properties.depProps.viewId,
                listLabel: strings.SelectList,
                viewLabel: strings.SelectView,
                onPropertyChange: this.onPropertyChange
              })
            ]
          }
        ]
      }
    ]
  };
}

{% endhighlight %}
</div>
<span style="background-color: white; color: #666666; font-family: &quot;trebuchet ms&quot; , &quot;trebuchet&quot; , &quot;verdana&quot; , sans-serif; font-size: 13.2px;">Now we need to implement UI for the Field and logic to get data from our data helpers. In this post I'll show example of simple </span><span style="background-color: white; color: #666666; font-family: &quot;courier new&quot; , &quot;courier&quot; , monospace; font-size: 13.2px;">select</span><span style="background-color: white; color: #666666; font-family: &quot;trebuchet ms&quot; , &quot;trebuchet&quot; , &quot;verdana&quot; , sans-serif; font-size: 13.2px;">&nbsp;elements without Office UI Fabric-like styling.</span><br /><span style="background-color: white; color: #666666; font-family: &quot;trebuchet ms&quot; , &quot;trebuchet&quot; , &quot;verdana&quot; , sans-serif; font-size: 13.2px;">I prefer to use Knockout.js as a framework (because I know it better than React and because looks like it is used for Lists and Libraries new experience).</span><br /><span style="background-color: white; color: #666666; font-family: &quot;trebuchet ms&quot; , &quot;trebuchet&quot; , &quot;verdana&quot; , sans-serif; font-size: 13.2px;">To use Knockout we need to install npm module:</span><br />
<div markdown="1">
{% highlight console %}
npm i knockout --save-dev

{% endhighlight %}
</div>
<span style="background-color: white; color: #666666; font-family: &quot;trebuchet ms&quot; , &quot;trebuchet&quot; , &quot;verdana&quot; , sans-serif; font-size: 13.2px;">And add path to knockout js file in config/config.json in </span><span style="background-color: white; color: #666666; font-family: &quot;courier new&quot; , &quot;courier&quot; , monospace; font-size: 13.2px;">externals</span><span style="background-color: white; color: #666666; font-family: &quot;trebuchet ms&quot; , &quot;trebuchet&quot; , &quot;verdana&quot; , sans-serif; font-size: 13.2px;">&nbsp;section:</span><br />
<div markdown="1">
{% highlight javascript %}
{
  "entries": [
    //...
  ],
  "externals": {
    //...
    "knockout": "node_modules/knockout/build/output/knockout-latest.js"
  },
  //...
}

{% endhighlight %}
</div>
<span style="background-color: white; color: #666666; font-family: &quot;trebuchet ms&quot; , &quot;trebuchet&quot; , &quot;verdana&quot; , sans-serif; font-size: 13.2px;"><br /></span><span style="background-color: white; color: #666666; font-family: &quot;trebuchet ms&quot; , &quot;trebuchet&quot; , &quot;verdana&quot; , sans-serif; font-size: 13.2px;">So we'll create Model, View and ViewModel for our field.</span><br /><span style="background-color: white; color: #666666; font-family: &quot;trebuchet ms&quot; , &quot;trebuchet&quot; , &quot;verdana&quot; , sans-serif; font-size: 13.2px;">Let's start with model. It should retrieve lists and views from data helpers and... that's it(/webparts/depProps/controls/PropertyPaneViewSelectorModel.ts):</span><br />
<div markdown="1">
{% highlight typescript %}
import {
  IWebPartContext
} from '@microsoft/sp-client-preview';
import { IDataHelper } from '../data-helpers/DataHelperBase';
import { DataHelpersFactory } from '../data-helpers/DataHelpersFactory';
import { ISPList, ISPView } from '../common/SPEntities';

/**
 * ViewSelector component model
 */
export class PropertyPaneViewSelectorModel {
  /**
   * data helper
   */
  private _dataHelper: IDataHelper;

  /**
   * ctor
   */
  constructor(_context: IWebPartContext) {
    this._dataHelper = DataHelpersFactory.createDataHelper(_context);
  }

  /**
   * Gets lists collection
   */
  public getLists(): Promise<ISPList[]> {
    return this._dataHelper.getLists();
  }

  /**
   * Gets views collection
   */
  public getViews(listId: string): Promise<ISPView[]> {
    return this._dataHelper.getViews(listId);
  }
}

{% endhighlight %}
</div>
<span style="background-color: white; color: #666666; font-family: &quot;trebuchet ms&quot; , &quot;trebuchet&quot; , &quot;verdana&quot; , sans-serif; font-size: 13.2px;">The view will contain 2 </span><span style="background-color: white; color: #666666; font-family: &quot;courier new&quot; , &quot;courier&quot; , monospace; font-size: 13.2px;">select</span><span style="background-color: white; color: #666666; font-family: &quot;trebuchet ms&quot; , &quot;trebuchet&quot; , &quot;verdana&quot; , sans-serif; font-size: 13.2px;">&nbsp;elements with bindings (in the GitHub repository the view code is updated to render Office UI Fabric-like markup that will be described in the next post):</span><br />
<div markdown="1">
{% highlight typescript %}
/**
 * ViewSelector component view
 */
export class PropertyPaneViewSelectorView {

  private _template: string = `
    <div>
      <select data-bind="options: lists, optionsText: 'Title', optionsValue: 'Id', value: currentList, optionsCaption: listLabel">
      </select>
    </div>
    <div>
      <select data-bind="options: views, optionsText: 'Title', optionsValue: 'Id', value: currentView, optionsCaption: viewLabel, enable: isListSelected"></select>
    </div>
  `;

  /**
   * Renders the HTML markup
   */
  public render(element: HTMLElement): Promise<void> {
    return new Promise<void>((resolve) => {
      element.innerHTML += this._template;
      resolve();
    });
  }
}

{% endhighlight %}
</div>
<span style="background-color: white; color: #666666; font-family: &quot;trebuchet ms&quot; , &quot;trebuchet&quot; , &quot;verdana&quot; , sans-serif; font-size: 13.2px;">And finally ViewModel that contains binded properties and logic to update UI and initiate web part update (in the Git repository the ViewModel code is updated to work with Office UI Fabric-like dropdowns that will be described in the next post):</span><br />
<div markdown="1">
{% highlight typescript %}
import {
  IWebPartContext
} from '@microsoft/sp-client-preview';
import { ISPList, ISPView } from '../common/SPEntities';
import { PropertyPaneViewSelectorModel } from './PropertyPaneViewSelectorModel';
import * as ko from 'knockout';
import {
  IPropertyPaneViewSelectorFieldPropsInternal
} from './Common';

import { IDropdownOption } from '../components/dropdown/viewmodel';

/**
 * ViewSelector component view model
 */
export class PropertyPaneViewSelectorViewModel {
  /**
   * Web part context
   */
  private _context: IWebPartContext;
  /**
   * Current List Id
   */
  private _listId: string;
  /**
   * Current View Id
   */
  private _viewId: string;
  /**
   * ViewSelector component properties
   */
  private _properties: IPropertyPaneViewSelectorFieldPropsInternal;

  /**
   * MVVM model
   */
  private _model: PropertyPaneViewSelectorModel;

  /**
   * Observable collection of lists
   */
  protected lists: KnockoutObservableArray<ISPList>;
  /**
   * Observable collection of views
   */
  protected views: KnockoutObservableArray<ISPView>;
  /**
   * Current selected list
   */
  protected currentList: KnockoutObservable<string>;
  /**
   * Current selected view
   */
  protected currentView: KnockoutObservable<string>;

  /**
   * Flag if there is a list selection
   */
  protected isListSelected: KnockoutObservable<boolean>;

  /**
   * List dropdown label
   */
  protected listLabel: string;
  /**
   * View dropdown label
   */
  protected viewLabel: string;

  /**
   * ctor
   * */
  public constructor(_properties: IPropertyPaneViewSelectorFieldPropsInternal) {
    this._properties = _properties;
    this._context = _properties.context;
    this._listId = _properties.listId;
    this._viewId = _properties.viewId;
    this.listLabel = _properties.listLabel;
    this.viewLabel = _properties.viewLabel;
    this.currentList = ko.observable<string>(this._listId);
    this.currentView = ko.observable<string>(this._viewId);
    this.isListSelected = ko.observable<boolean>(this._listId !== '-1');

    // subscribing on changes in lists dropdown
    this.currentList.subscribe((value) => {
      this._listId = value;
      this._initViews().then(() => {
        this.isListSelected(this._listId !== '-1');
      });

      this._firePropertyChange();

    });

    // subscribing on changes in view dropdown
    this.currentView.subscribe((value) => {
      this._viewId = value;
      this._firePropertyChange();
    });

    this.lists = ko.observableArray<ISPList>();
    this.views = ko.observableArray<ISPView>();

    this._model = new PropertyPaneViewSelectorModel(this._context);
  }

  /**
   * Fires property changed event handler
   * This method should be called in 'Reactive' mode of web part properties pane
   */
  private _firePropertyChange(): void {
    if (this._properties.onPropertyChange) {
      this._properties.onPropertyChange(this._properties.targetProperty, { listId: this._listId || '-1', viewId: this._viewId || '-1' });
    }
  }

  /**
   * Initializes the view model
   */
  public init(): Promise>void< {
    return new Promise>void<((resolve) => {
      this._model.getLists().then((lists) => {  // getting lists
        this.lists(lists);
        this._initViews().then(() => {
          resolve();
        });
      });
    });
  }

  /**
   * Initializes views collection based on selected list
   */
  private _initViews(): Promise<void> {
    return new Promise<void>((resolve) => {
      if (this._listId) {
        this._model.getViews(this._listId).then((views) => { // getting views
          this.views(views);
          resolve();
        });
      }
      else
        resolve();
    });
  }
}

{% endhighlight %}
</div>
<span style="background-color: white; color: #666666; font-family: &quot;trebuchet ms&quot; , &quot;trebuchet&quot; , &quot;verdana&quot; , sans-serif; font-size: 13.2px;"><br /></span><span style="background-color: white; color: #666666; font-family: &quot;trebuchet ms&quot; , &quot;trebuchet&quot; , &quot;verdana&quot; , sans-serif; font-size: 13.2px;">Final step is to call View's rendering in Field </span><span style="background-color: white; color: #666666; font-family: &quot;courier new&quot; , &quot;courier&quot; , monospace; font-size: 13.2px;">render</span><span style="background-color: white; color: #666666; font-family: &quot;trebuchet ms&quot; , &quot;trebuchet&quot; , &quot;verdana&quot; , sans-serif; font-size: 13.2px;">&nbsp;method, initialize ViewModel and apply Knockout bindings:</span><br />
<div markdown="1">
{% highlight typescript %}
/**
 * Rendering the component
 */
private _render(elem: HTMLElement): void {
  this._view.render(elem).then(() => {     // rendering of HTML markup
    this._viewModel.init().then(() => {    // then getting data
      ko.applyBindings(this._viewModel, elem);   // then applying bindings
    });
  });
}

{% endhighlight %}
</div>
<span style="background-color: white; color: #666666; font-family: &quot;trebuchet ms&quot; , &quot;trebuchet&quot; , &quot;verdana&quot; , sans-serif; font-size: 13.2px;"><br /></span><span style="background-color: white; color: #666666; font-family: &quot;trebuchet ms&quot; , &quot;trebuchet&quot; , &quot;verdana&quot; , sans-serif; font-size: 13.2px;"><br /></span><span style="background-color: white; color: #666666; font-family: &quot;trebuchet ms&quot; , &quot;trebuchet&quot; , &quot;verdana&quot; , sans-serif; font-size: 13.2px;"><br /></span><span style="background-color: white; color: #666666; font-family: &quot;trebuchet ms&quot; , &quot;trebuchet&quot; , &quot;verdana&quot; , sans-serif; font-size: 13.2px;">Now you can see the result:</span><br /><div class="separator" style="clear: both; text-align: center;"><a href="https://4.bp.blogspot.com/-knaDamkqn38/WBEtgYYwjbI/AAAAAAAAAaM/qUGS9JCUPxsXle4gnDppXnnZRfPMLsgsACLcB/s1600/Screen%2BShot%2B2016-10-26%2Bat%2B3.20.50%2BPM.png" imageanchor="1" style="margin-left: 1em; margin-right: 1em;"><img border="0" height="265" src="https://4.bp.blogspot.com/-knaDamkqn38/WBEtgYYwjbI/AAAAAAAAAaM/qUGS9JCUPxsXle4gnDppXnnZRfPMLsgsACLcB/s640/Screen%2BShot%2B2016-10-26%2Bat%2B3.20.50%2BPM.png" width="640" /></a></div><span style="background-color: white; color: #666666; font-family: &quot;trebuchet ms&quot; , &quot;trebuchet&quot; , &quot;verdana&quot; , sans-serif; font-size: 13.2px;">There is a lot of code here in the post. But there is nothing really difficult to understand. Let me know if you have any questions or comments.</span><br /><span style="background-color: white; color: #666666; font-family: &quot;trebuchet ms&quot; , &quot;trebuchet&quot; , &quot;verdana&quot; , sans-serif; font-size: 13.2px;">See you in next post.</span><br /><span style="background-color: white; color: #666666; font-family: &quot;trebuchet ms&quot; , &quot;trebuchet&quot; , &quot;verdana&quot; , sans-serif; font-size: 13.2px;"><br /></span><span style="background-color: white; color: #666666; font-family: &quot;trebuchet ms&quot; , &quot;trebuchet&quot; , &quot;verdana&quot; , sans-serif; font-size: 13.2px;">Have fun!</span><span style="background-color: white; color: #666666; font-family: &quot;trebuchet ms&quot; , &quot;trebuchet&quot; , &quot;verdana&quot; , sans-serif; font-size: 13.2px;"><br /></span><span style="background-color: white; color: #666666; font-family: &quot;trebuchet ms&quot; , &quot;trebuchet&quot; , &quot;verdana&quot; , sans-serif; font-size: 13.2px;"><br /></span><span style="background-color: white; color: #666666; font-family: &quot;trebuchet ms&quot; , &quot;trebuchet&quot; , &quot;verdana&quot; , sans-serif; font-size: 13.2px;"><br /></span></div><br /></div>