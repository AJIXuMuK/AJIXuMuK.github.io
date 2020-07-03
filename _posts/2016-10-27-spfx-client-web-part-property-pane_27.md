---
layout: post
title: 'SPFx. Client Web Part Property Pane Dependent Properties. Part III: Styling
  Dependent Properties Control as Office UI Fabric dropdowns. Knockout version.'
date: '2016-10-27T22:18:00.000-07:00'
author: Alex Terentiev
tags:
- JavaScript
- SharePoint Online
- SPFx
- Client Web Part
- SharePoint Framework
- Future of SharePoint
- SharePoint
modified_time: '2016-10-28T12:51:05.676-07:00'
thumbnail: assets/images/posts/2016/2016-10-27-1.png
featured_image: assets/images/posts/2016/2016-10-27-1.png
blogger_id: tag:blogger.com,1999:blog-3066084330774405472.post-7269803676887840015
blogger_orig_url: http://blog.aterentiev.com/2016/10/spfx-client-web-part-property-pane_27.html
---

This is the last post of three that explain how to add dependent properties to Client Web Part Property Pane.<br />As an example of such properties I decided to use Lists dropdown and Views dropdown. Views dropdown is populated based on selected List. As a result of these three posts we'll have a fully working web part with ability to select List View in Property Pane:<br /><div class="separator" style="clear: both; text-align: center;"><img border="0" src="{{site.baseurl}}/assets/images/posts/2016/2016-10-27-1.png" /></div><br />Here is a content of 3 posts:<br /><ol><li><a href="http://tricky-sharepoint.blogspot.com/2016/10/spfx-client-web-part-property-pane.html" target="_blank">SPFx. Client Web Part Property Pane Dependent Properties. Part I: Preparation.</a></li><li><a href="http://tricky-sharepoint.blogspot.com/2016/10/spfx-client-web-part-property-pane_26.html" target="_blank">SPFx. Client Web Part Property Pane Dependent Properties. Part II: Dependent Properties Control.</a></li><li>SPFx. Client Web Part Property Pane Dependent Properties. Part III: Styling Dependent Properties Control as Office UI Fabric dropdowns. Knockout version. (current post)</li></ol><div>All the code is available on GitHub (<a href="https://github.com/AJIXuMuK/SPFx/tree/master/dep-props-custom-class" target="_blank">https://github.com/AJIXuMuK/SPFx/tree/master/dep-props-custom-class</a>)<br />I'm using Visual Studio Code IDE for the tutorial.<br /><a name='more'></a><br />This post describes creation of custom Knockout bindings, custom Knockout components and styling your components in Office UI Fabric manner.<br /><br />In previous post we created a custom field that consists of two dependent dropdowns. Everything works good but looks not so good. It would be much better if our dropdowns look similar to <a href="https://dev.office.com/fabric#/components/dropdown" target="_blank">Office UI Fabric Dropdown</a>. Unfortunately there is no Knockout version of Office UI Fabric Dropdown and a Dropdown from Fabric JS is pretty static - it creates <span style="font-family: &quot;courier new&quot; , &quot;courier&quot; , monospace;">&lt;ul&gt;</span>&nbsp;markup based on a <span style="font-family: &quot;courier new&quot; , &quot;courier&quot; , monospace;">&lt;select&gt;.</span>&nbsp;But if you change something in your select dynamically it won't reflect UL until you recreate the Fabric JS dropdown.<br />So let's create our own Knockout.js component styled as Office UI Fabric Dropdown. You can find documentation on Knockout components <a href="http://knockoutjs.com/documentation/component-overview.html" target="_blank">here</a>&nbsp;so I won't spend time to describe it deeply. In few words we need to create a view, a viewmodel and register the component to be able to use it in our app.<br />First of all, we need to create some interface that will describe a single dropdown item. We need value, text and selected flag to describe an item (/webparts/depProps/components/dropdown/viewmodel): <br />
<div markdown="1">
{% highlight typescript %}
/**
 * Dropdown option item
 */
export interface IDropdownOption {
  value: string | number;
  text: string;
  selected: boolean;
}

{% endhighlight %}
</div>
<br />It also would be great if our component's bindings look similar to standard <span style="font-family: &quot;courier new&quot; , &quot;courier&quot; , monospace;">&lt;select&gt;</span>&nbsp;bindings. It means that we could provide<br /><ul><li>options - observable array of dropdown items</li><li>optionsCaption - dummy option to prefix a list of dropdown items</li><li>value - selected value</li><li>disabled - flag if a dropdown disabled or not</li></ul><div>in a manner like</div>
<div markdown="1">
{% highlight html %}
<our-component params="options: listOfOptions, optionsCaption: caption, value: currentValue, disabled: isDisabled"></our-component>

{% endhighlight %}
</div>
<div>Based on the above we can create an interface that describes parameters of our component (/webparts/depProps/components/dropdown/viewmodel): <br />
<div markdown="1">
{% highlight typescript %}
/**
 * Parameters that can be provided to Dropdown component
 */
export interface IDropdownViewModelParams {
  /**
   * options collection as KnockoutObservableArray
   */
  options: KnockoutObservableArray<idropdownoption>;
  /**
   * Dropdown empty element text and text of Dropdown label
   */
  optionsCaption: string;
  /**
   * Selected value
   */
  value: KnockoutObservable<string>;

  /**
   * Is Dropdown disabled
   */
  disabled?: KnockoutObservable<boolean>;
}
</boolean></string></idropdownoption>
{% endhighlight %}
</div>
<br /></div><div>Now let's look at Office UI Fabric Dropdown markup to understand what we need to implement in our component's view:</div>
<div markdown="1">
{% highlight html %}
<div>
  <label class="ms-Label">Label goes here</label>
  <div class="ms-Dropdown">
    <span class="ms-Dropdown-title">Selected Option</span>
    <i class="ms-Dropdown-caretDown ms-Icon ms-Icon--ChevronDown"></i>
    <ul class="ms-Dropdown-items">
      <li class="ms-Dropdown-item" role="option">Option 1</li>
      <li class="ms-Dropdown-item">Option 2</li>
      <!-- etc. -->
    </ul>
  </div>
</div>

{% endhighlight %}
</div>
<div><br /></div>This is how our view should look. LI elements will be created dynamically based on <span style="font-family: &quot;courier new&quot; , &quot;courier&quot; , monospace;">options</span>&nbsp;property. The only thing we need to decide is how to implement inserting of <span style="font-family: &quot;courier new&quot; , &quot;courier&quot; , monospace;">optionsCaption</span>&nbsp;dummy item before <span style="font-family: &quot;courier new&quot; , &quot;courier&quot; , monospace;">options</span>&nbsp;in the dropdown. There are two ways for that. First one is to implement the logic in viewmodel: get <span style="font-family: &quot;courier new&quot; , &quot;courier&quot; , monospace;">options</span>&nbsp;array and insert a new element there which will have <span style="font-family: &quot;courier new&quot; , &quot;courier&quot; , monospace;">optionsText</span>&nbsp;as text, -1 as value. And in the view we can use simple <span style="font-family: &quot;courier new&quot; , &quot;courier&quot; , monospace;">for</span>&nbsp;binding to iterate through the items and create <span style="font-family: &quot;courier new&quot; , &quot;courier&quot; , monospace;">li</span>&nbsp;elements for each of them.<br />Second one is to create a <a href="http://knockoutjs.com/documentation/custom-bindings.html" target="_blank">custom binding</a>&nbsp;similar to out-of-the-box <span style="font-family: &quot;courier new&quot; , &quot;courier&quot; , monospace;">options</span>&nbsp;binding.<br />I selected the second option to practice in custom bindings.<br />Let's call the binding class as <span style="font-family: &quot;courier new&quot; , &quot;courier&quot; , monospace;">MsDropdownOptions</span>&nbsp;and put it in /webparts/depProps/bindings/MsDropdownOptions.ts: <br />
<div markdown="1">
{% highlight typescript %}
import * as ko from 'knockout';
/**
 * Knockout 'msoptions' binding.
 * Allows to bind Fabric UI dropdown options in the similar way to standard select options binding
 */
class MsDropdownOptions {
  constructor() {
    this.init = this.init.bind(this);
    this.update = this.update.bind(this);
  }

  /**
   * This will be called when the binding is first applied to an element
   */
  public init(element: any, valueAccessor: () => any, allBindingsAccessor?: KnockoutAllBindingsAccessor, viewModel?: any, bindingContext?: KnockoutBindingContext): void | { controlsDescendantBindings: boolean; } {
    // check if binding is applied to the correct element
    if (element.tagName.toLowerCase() !== 'div' || element.className.indexOf('ms-Dropdown') === -1)
      throw new Error('msoptions binding applies only to <div class="ms-Dropdown"> elements');

    // removing previous content if any
    this.removePreviousContent(element);

    return { 'controlsDescendantBindings': true };
  }
  /**
   * Removes previous content of dropdown
   */
  private removePreviousContent(element: any) {
    const titleEl: HTMLSpanElement = element.querySelector('span');
    if (titleEl) {
      titleEl.textContent = '';
    }

    const listEl = element.querySelector('ul');
    if (!listEl)
      throw new Error('Incorrect markup in ms-Dropdown element');

    while (listEl.children.length) {
      listEl.children[0].remove();
    }
  }

  /**
   * This method adds all the options and also updates selected item and text
   */
  private addNewContent(element: any, options: {}[], selectedValue: KnockoutObservable<string>, itemSelected: (evt: MouseEvent) => any): void {
    const titleEl: HTMLSpanElement = element.querySelector('span');
    const listEl = element.querySelector('ul');
    let selectedValueUnwrapped: string = selectedValue &amp;&amp; ko.utils.unwrapObservable(selectedValue);
    let selectedValueChanged: boolean = false;

    if (!listEl || !titleEl)
      throw new Error('Incorrect markup in ms-Dropdown element');

    for (let i: number = 0, len: number = options.length; i < len; i++) {
      const liEl: HTMLLIElement = document.createElement('li');
      const option = options[i];
      liEl.textContent = option['text'];
      liEl.setAttribute('aria-value', option['value']);
      liEl.setAttribute('role', 'option');
      liEl.setAttribute('aria-text', option['text']);
      liEl.className = 'ms-Dropdown-item';

      let isSelected: boolean = false;

      if (selectedValueUnwrapped &amp;&amp; selectedValueUnwrapped === option['value'])
        isSelected = true;
      else {
        isSelected = option['selected'] === true || option['selected'] === true || option['selected'] === 'selected';
        if (isSelected) {
          selectedValueUnwrapped = option['value'];
          selectedValueChanged = true;
        }
      }

      liEl.setAttribute('aria-selected', isSelected + '');

      if (isSelected) {
        titleEl.textContent = option['text'];
        liEl.className += ' is-selected';
      }

      if (itemSelected) {
        liEl.addEventListener('click', itemSelected); // itemSelected is provided from binding as well
      }

      listEl.appendChild(liEl);
    }

    if (!titleEl.textContent &amp;&amp; options.length > 0) {
      titleEl.textContent = options[0]['text'];
      listEl.children[0].setAttribute('aria-selected', 'true');
      selectedValueUnwrapped = options[0]['value'];
      selectedValueChanged = true;
    }

    if (selectedValueChanged &amp;&amp; selectedValue)
      selectedValue(selectedValueUnwrapped);
  }

  /**
   * This will be called once when the binding is first applied to an element,
   * and again whenever any observables/computeds that are accessed change
   */
  public update(element: any, valueAccessor: () => any, allBindingsAccessor?: KnockoutAllBindingsAccessor,
    viewModel?: any, bindingContext?: KnockoutBindingContext): void {
    this.removePreviousContent(element);

    let unwrappedArray: any = ko.utils.unwrapObservable(valueAccessor());
    let captionValue: any;
    let filteredArray: {}[] = [];
    let itemSelected: (evt: MouseEvent) => any;

    if (unwrappedArray) {
      if (typeof unwrappedArray.length == "undefined") // Coerce single value into array
        unwrappedArray = [unwrappedArray];

      // Filter out any entries marked as destroyed
      filteredArray = ko.utils.arrayFilter(unwrappedArray, (item) => {
        return item === undefined || item === null || !ko.utils.unwrapObservable(item['_destroy']);
      });

      // If caption is included, add it to the array
      if (allBindingsAccessor['has']('optionsCaption')) {
        captionValue = ko.utils.unwrapObservable(allBindingsAccessor.get('optionsCaption'));
        // If caption value is null or undefined, don't show a caption
        if (captionValue !== null &amp;&amp; captionValue !== undefined) {
          filteredArray.unshift({ value: "-1", text: captionValue });
        }
      }
    } else {
      // If a falsy value is provided (e.g. null), we'll simply empty the select element
    }

    if (allBindingsAccessor['has']('itemSelected'))
      itemSelected = allBindingsAccessor.get('itemSelected') as (evt: MouseEvent) => any;
    this.addNewContent(element, filteredArray, allBindingsAccessor.get('value'), itemSelected);
  }
}

{% endhighlight %}
</div>
Also we need to register this binding inside knockout. For that we need to extend <span style="font-family: &quot;courier new&quot; , &quot;courier&quot; , monospace;">KnockoutBindingHandlers</span>&nbsp;interface definition with a new property (/webparts/depProps/bindings/bindings.d.ts): <br />
<div markdown="1">
{% highlight typescript %}
/// <reference path="../../../../typings/knockout/knockout.d.ts">
interface KnockoutBindingHandlers {
  msoptions: KnockoutBindingHandler; // custom binding for Knockout Fabric UI dropdown options
}
</reference>
{% endhighlight %}
</div>
<br />And also we'll create a function that will register our custom binding (/webparts/depProps/bindings/MsDropdownOptions.ts): <br />
<div markdown="1">
{% highlight typescript %}
/**
 * This API adds custom 'msoptions' binding to Knockout bindingHandlers object
 */
export function addMsDropdownBindingHandler(): void {
  if (!ko.bindingHandlers.msoptions)
    ko.bindingHandlers.msoptions = new MsDropdownOptions();
}

{% endhighlight %}
</div>
Now we can use this binging in our view. So the final markup with bindings (based on Office UI Fabric Dropdown markup) will look like (/webparts/depProps/components/dropdown/view.ts): <br />
<div markdown="1">
{% highlight typescript %}
/**
 * Dropdown markup (Fabric UI style)
 */
export default class DrowdownView {
  public static templateHtml: string = `
    <div>
      <label class="ms-label" data-bind="text: optionsCaption"></label>
      <div class="ms-Dropdown" data-bind="click: onOpenDropdown, msoptions: options, optionsCaption: optionsCaption, value: value, itemSelected: onItemSelected, css: { 'is-open': isOpen(), 'is-disabled': disabled() }">
        <span class="ms-Dropdown-title" data-bind=""></span>
        <i class="ms-Dropdown-caretDown ms-Icon ms-Icon--ChevronDown"></i>
        <ul class="ms-Dropdown-items" role="listbox">
        </ul>
      </div>
    </div>
  `;
}

{% endhighlight %}
</div>
<span style="font-family: &quot;courier new&quot; , &quot;courier&quot; , monospace;">msoptions, optionsCaption</span>&nbsp;and <span style="font-family: &quot;courier new&quot; , &quot;courier&quot; , monospace;">itemSelected</span>&nbsp;parameters are handled inside created custom binding.<br /><br />As you can see there are some methods in the binding (<span style="font-family: &quot;courier new&quot; , &quot;courier&quot; , monospace;">onOpenDropdown, onItemSelected, isOpen, disabled</span>). All these methods should be implemented in a viewmodel. So let's create the viewmodel (/webpars/depProps/components/dropdown/viewmodel.ts). <br />
<div markdown="1">
{% highlight typescript %}
import * as ko from 'knockout';

/**
 * ViewModel of custom Dropdown component
 */
export class DropdownViewModel {
  /**
   * Last opened Dropdown ViewModel
   */
  private static _openedDropdownVM: DropdownViewModel;
  /**
   * Component initial parameters
   */
  private _params: IDropdownViewModelParams;

  /**
   * Dropdown options
   */
  protected options: KnockoutObservableArray<IDropdownOption>;
  /**
   * Options caption
   */
  protected optionsCaption: string;
  /**
   * Current selected value
   */
  protected value: KnockoutObservable<string>;
  /**
   * Is Dropdown opened
   */
  protected isOpen: KnockoutObservable<boolean>;

  /**
   * Is Dropdown disabled
   */
  protected disabled: KnockoutObservable<boolean>;

  /**
   * ctor
   */
  constructor(params: IDropdownViewModelParams) {
    this._params = params;
    this.options = params.options;
    this.optionsCaption = params.optionsCaption;
    this.value = params.value;

    if (params.disabled)
      this.disabled = params.disabled;
    else
      this.disabled = ko.observable<boolean>(false);

    // initally dropdown is closed
    this.isOpen = ko.observable<boolean>(false);

    //
    // binding all handlers to current instance
    //
    this.onItemSelected = this.onItemSelected.bind(this);
    this.onOpenDropdown = this.onOpenDropdown.bind(this);
    this._onDocClick = this._onDocClick.bind(this);
  }

  /**
   * item selected handler
   */
  public onItemSelected(evt: MouseEvent) {
    var selectedItem = evt.srcElement;
    this.value(selectedItem.getAttribute('aria-value'));
  }

  /**
   * Open-close dropdown handler
   */
  public onOpenDropdown(vm: DropdownViewModel, evt: MouseEvent) {
    if (this.disabled())
      return;
    const isOpen: boolean = ko.utils.unwrapObservable(this.isOpen);
    evt.stopPropagation();
    this.isOpen(!isOpen);

    if (!isOpen) {
      if (DropdownViewModel._openedDropdownVM &amp;&amp; DropdownViewModel._openedDropdownVM !== this) {
        DropdownViewModel._openedDropdownVM._onDocClick(null);
      }

      DropdownViewModel._openedDropdownVM = this;
      document.addEventListener('click', this._onDocClick);
    }
  }

  /**
   * document.onclick handler
   */
  private _onDocClick(evt: MouseEvent) {
    this.isOpen(false);
    document.removeEventListener('click', this._onDocClick);
  }
}

{% endhighlight %}
</div>
Now register it (/webparts/depProps/components/dropdown/Dropdown.ts): <br />
<div markdown="1">
{% highlight typescript %}
import DropdownView from './view';
import { DropdownViewModel } from './viewmodel';
import * as ko from 'knockout';
import { addMsDropdownBindingHandler } from '../../bindings/MsDropdownOptions';

/**
 * custom dropdown component name
 */
export const DROPDOWN_COMPONENT: string = 'spfx_dropdown';

/**
 * API to register custom dropdown component
 */
export function registerDropdown(): boolean {
  if (!ko.bindingHandlers.msoptions)
    addMsDropdownBindingHandler();

  if (!ko.components.isRegistered(DROPDOWN_COMPONENT)) {
    ko.components.register(DROPDOWN_COMPONENT, {
      template: DropdownView.templateHtml,
      viewModel: DropdownViewModel
    });
  }

  return true;
}

{% endhighlight %}
</div>
<br />And finally use it in our custom Property Pane field. For that:<br /><ul><li>import component name in&nbsp;PropertyPaneViewSelectorView.ts<br />
<div markdown="1">
{% highlight typescript %}
import { DROPDOWN_COMPONENT } from '../components/dropdown/Dropdown';

{% endhighlight %}
</div>
</li><li>change the <span style="font-family: &quot;courier new&quot; , &quot;courier&quot; , monospace;">_template</span>&nbsp;variable of <span style="font-family: &quot;courier new&quot; , &quot;courier&quot; , monospace;">PropertyPaneViewSelectorView</span>&nbsp;class to:<br />
<div markdown="1">
{% highlight typescript %}
private _template: string = `
    <div class="view-selector-component">
      <${DROPDOWN_COMPONENT} params="options: lists, optionsCaption: listLabel, value: currentList"></${DROPDOWN_COMPONENT}>
      <${DROPDOWN_COMPONENT} params="options: views, optionsCaption: viewLabel, value: currentView, disabled: noListSelection"></${DROPDOWN_COMPONENT}>
    </div>
  `;

{% endhighlight %}
</div>
</li><li>and change <span style="font-family: &quot;courier new&quot; , &quot;courier&quot; , monospace;">PropertyPaneViewSelector</span>&nbsp;class to register the component in constructor:<br />
<div markdown="1">
{% highlight typescript %}
// somewhere at the beginning of the file
import { registerDropdown } from '../components/dropdown/Dropdown';

/**
 * ctor
 */
 public constructor(_targetProperty: string, _properties: IPropertyPaneViewSelectorFieldPropsInternal) {
   // add this to the end of the constructor method
   // registering used custom Knockout components (dropdown)
   this._registerComponents();
 }

/**
 * Registers used custom Knockout components
 */
private _registerComponents(): void {
  registerDropdown();
}

{% endhighlight %}
</div>
</li></ul><div>Now we finished and can enjoy our beautiful custom Property Pane Field with Office UI Fabric-like dropdowns:<br /><div class="separator" style="clear: both; text-align: center;"><img border="0" src="{{site.baseurl}}/assets/images/posts/2016/2016-10-27-2.png" /></div>Let me know if you have any questions or comments.<br /><br />Have fun!</div></div>