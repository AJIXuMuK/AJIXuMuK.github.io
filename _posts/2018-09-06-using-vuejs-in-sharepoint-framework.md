---
layout: post
title: 'Using Vue.js in SharePoint Framework Applications. Part IV: Web Part Property
  Pane Control'
date: '2018-09-06T21:32:00.000-07:00'
author: Alex Terentiev
tags:
- SharePoint Online
- Client Side Web Part
- SPFx
- Vuejs
- Vue
- yo
- Yeoman
- generator-sharepoint
- O365
- SharePoint Framework
- Office 365
modified_time: '2018-10-07T12:14:28.295-07:00'
thumbnail: https://2.bp.blogspot.com/-W1g5ejpWAM4/W5H_JGBKZwI/AAAAAAAAA4Y/euMr9XhpgjUCNWa652dPkAr_b4hQNnlYwCLcBGAs/s72-c/Screen%2BShot%2B2018-09-06%2Bat%2B9.08.43%2BPM.png
blogger_id: tag:blogger.com,1999:blog-3066084330774405472.post-6247888011485655552
blogger_orig_url: http://blog.aterentiev.com/2018/09/using-vuejs-in-sharepoint-framework.html
---

This is the fourth post about SharePoint Framework and Vue.js. In this post I want to go through the process of creation custom Property Pane control using Vue.js.<br /><br />List of posts:<br /><ol><li><a href="http://blog.aterentiev.com/2018/04/using-vuejs-in-sharepoint-framework.html" target="_blank">Whats and Whys</a></li><li><a href="http://blog.aterentiev.com/2018/05/using-vuejs-in-sharepoint-framework.html" target="_blank">Default SPFx web part using Vue.js</a></li><li><a href="http://blog.aterentiev.com/2018/07/using-vuejs-in-sharepoint-framework.html" target="_blank">Yeoman generator with Vue support</a></li><li>Web Part Property Pane Control (this post)</li><li><a href="http://blog.aterentiev.com/2018/09/using-vuejs-in-sharepoint-framework_17.html" target="_blank">Use React Components inside Vue.js Solution</a></li></ol><b>Code: </b><a href="https://github.com/AJIXuMuK/vuejs/tree/master/proppane-control" target="_blank">https://github.com/AJIXuMuK/vuejs/tree/master/proppane-control</a>.<br /><a name='more'></a>In previous posts we discussed what steps are needed to make Vue.js work in SPFx projects and how to use Vue.js Single File Components inside SharePoint Framework Web Parts and Extensions.<br />I've also mentioned <a href="https://www.npmjs.com/package/generator-vuespfx" target="_blank">Yeoman VueSpfx generator</a> that does all needed configurations.<br />In the current post I want to discuss how to create custom controls for Web Part Property Pane.<br /><h2>Files and Structure</h2>If you look at <a href="https://docs.microsoft.com/en-us/sharepoint/dev/spfx/web-parts/guidance/build-custom-property-pane-controls" target="_blank">official documentation</a> and <a href="https://github.com/SharePoint/sp-dev-fx-property-controls" target="_blank">Reusable SPFx Property Pane Controls repo</a> you'll notice that there is a "best practice" on how to structure files and code of custom controls for property pane when you're going to use some JavaScript framework.<br /><b>Note: I'm using "component" and "control" terms here. The "control" is for property pane custom control that we're creating. The "component" is for framework-specific implementation of the control.</b><br />Let's say you want to create new <span class="code">ImageUrl</span> control which is a simple input (text box). In that case the file structure and contents will look like that: <ol><li><span class="code">IPropertyFieldImageUrl.ts</span> - file to contain custom control properties interface ("public" properties) that will be passed from the web part. This file also usually contains "internal" properties interface that extends "public" properties with <span class="code">IPropertyPaneCustomFieldProps</span> interface.<br />"Public" properties interface usually contains: <ul><li><span class="code">key</span> - a UNIQUE key indicates the identity of this control.</li><li><span class="code">properties</span> - parent Web Part properties</li><li><span class="code">onPropertyChange</span> - defines an onPropertyChange function to raise when the selected value changes. Normally this function must be defined with the 'this.onPropertyChange' method of the web part object.</li></ul>The "template" for the file looks like that: 
<div markdown="1">
{% highlight typescript %}

import { IPropertyPaneCustomFieldProps } from '@microsoft/sp-webpart-base';

export interface IPropertyFieldImageUrlProps {
  key: string;
  properties: any;
  onPropertyChange(propertyPath: string, oldValue: any, newValue: any): void;
  // other needed properties
}

export interface IPropertyFieldImageUrlPropsInternal extends IPropertyFieldImageUrlProps, IPropertyPaneCustomFieldProps {
}

{% endhighlight %}
</div>
</li><li><span class="code">IPropertyfieldImageUrlHost.ts</span> - file with interfaces that are needed for the component's functioning. In case of React it will contain Props and State interfaces.<br />For Vue.js we can define Props interface in this file. That interface will be used as a "contract" for our SFC. </li><li><span class="code">PropertyFieldImageUrl.ts</span> - custom control "entry point". It contains <ul><li>A function to create the control (this function will be used in <span class="code">getPropertyPaneConfiguration</span> method of the web part).</li><li>A "builder" class that is needed to correctly initialize, render and "orchestrate" the component. This class is instantiated from the function mentioned above.</li></ul>The "template" for that file looks like that: 
<div markdown="1">
{% highlight typescript %}

import { IPropertyPaneField, PropertyPaneFieldType } from '@microsoft/sp-webpart-base';
import {
    IPropertyFieldImageUrlProps,
    IPropertyFieldImageUrlPropsInternal
} from './IPropertyFieldImageUrl';
// other imports to reference component and its properties
class PropertyFieldImageUrlBuilder implements IPropertyPaneField<IPropertyFieldImageUrlProps> {
// rendering, correct initialization, and so on...
}
export function PropertyFieldImageUrl(targetProperty: string, properties: IPropertyFieldImageUrlProps): IPropertyPaneField<IPropertyFieldImageUrlProps> {
 return new PropertyFieldImageUrlBuilder(targetProperty, properties);
}

{% endhighlight %}
</div>
</li><li><span class="code">PropertyFieldImageUrlHost.module.scss</span> - SASS file with all the styles used in the component. </li><li>Framework-specific files for component implementation. In case of React it will be <span class="code">PropertyFieldImageUrlHost.tsx</span>. For Vue.js we'll add <span class="code">PropertyFieldImageUrlHost.vue</span> and remove <span class="code">PropertyFieldImageUrlHost.module.scss</span> as all the styles will be included in <span class="code">.vue</span> file. </li></ol><h2>Implementation</h2>Now, let's implement the control!<br />General idea of the control is to provide a text box with label that will allow user to enter image url (for example, SharePoint document url) as a simple text. Later, this value could be used in the Web Part to render the image.<br />It means that we'll need to have to additional properties in "public" properties interface: <ul><li><span class="code">label</span> - to provide the label for the text box. It could be hardcoded or received directly from web part's strings. But let's pass it from the web part.</li><li><span class="code">value</span> - the value of the text box. This value could be stored in web part's properties.</li></ul>It would be also great to use Office UI Fabric styles to render the control.<br />I'll be describing the implementation based on the files mentioned above.<br /><h3>IPropertyFieldImageUrl.ts</h3>As mentioned above, this file contains "public" and "internal" properties of the control.<br />Let's extend the "public" properties interface to include <span class="code">label</span> and <span class="code">value</span> properties. The "internal" properties interface will remain without changes. 
<div markdown="1">
{% highlight typescript %}

import { IPropertyPaneCustomFieldProps } from '@microsoft/sp-webpart-base';

/**
 * Public properties of the PropertyField custom field
 */
export interface IPropertyFieldImageUrlProps {
    /**
    * An UNIQUE key indicates the identity of this control
    */
    key: string;
    /**
     * Property field label displayed on top
     */
    label: string;
    /**
    * Parent Web Part properties
    */
    properties: any;
    /**
     * Defines an onPropertyChange function to raise when the items order changes.
     * Normally this function must be defined with the 'this.onPropertyChange'
     * method of the web part object.
     */
    onPropertyChange(propertyPath: string, oldValue: any, newValue: any): void;
    /**
     * current value
     */
    value?: string;
}

export interface IPropertyFieldImageUrlPropsInternal extends IPropertyFieldImageUrlProps, IPropertyPaneCustomFieldProps {
}

{% endhighlight %}
</div>
<h3>IPropertyfieldImageUrlHost.ts</h3>This file contains Props interface for Vue.js component. Here we should define all the properties that are passed from the "builder" class. In our case, we need to pass current value, label, unique key and handler to call when the value has been changed in the component: 
<div markdown="1">
{% highlight typescript %}

/**
 * PropertyFieldHost component props
 */
export interface IPropertyFieldImageUrlHostProps {
    label: string;
    value?: string;
    onValueChanged: (value: string) => void;
    /**
     * we're using uniqueKey instead of key because key is a "reserved" attribute
     */
    uniqueKey: string;
}

{% endhighlight %}
</div>
<h3>PropertyFieldImageUrl.vue - &lt;style&gt; Section</h3>The section "duplicates" the styles and hierarchy that is used in <span class="code">TextField</span> <a href="https://developer.microsoft.com/en-us/fabric#/components/textfield" target="_blank">Office UI Fabric component</a>. It also uses "theme" variables for border colors (you can read more about theme variables <a href="https://docs.microsoft.com/en-us/sharepoint/dev/spfx/use-theme-colors-in-your-customizations" target="_blank">here</a>, <a href="http://blog.aterentiev.com/2017/04/using-sharepoint-themes-colors-in-spfx.html" target="_blank">here</a>, and <a href="https://n8d.at/blog/how-to-use-theme-colors-in-spfx-web-parts/" target="_blank">here</a>). 
<div markdown="1">
{% highlight css %}

<style lang="scss" module>
@import '~@microsoft/sp-office-ui-fabric-core/dist/sass/_SPFabricCore.scss';

$inputBorder: "[theme:inputBorder, default: #a6a6a6]";
$inputBorderHovered: "[theme:inputBorderHovered, default: #212121]";

.ImageUrl {
    .label {
        display: block;
        padding: 5px 0;
    }

    .inputWrapper {
        border: 1px solid;
        border-color: $inputBorder;
        height: 32px;
        display: flex;

        &:hover {
            border-color: $inputBorderHovered;
        }

        & > input {
            border: none;
            padding: 0 12px;
            width: 100%;
        }
    }
}
</style>

{% endhighlight %}
</div>
<h3>PropertyFieldImageUrlHost.vue - &lt;script&gt; Section</h3>The section contains the logic of Vue.js component.<br />In our simple example we need multiple things here: <ul><li>Implement <span class="code">IPropertyFieldImageUrlHost</span> interface as Vue.js "props". Because I'm using <span class="code">vue-property-decorator</span>(<a href="https://www.npmjs.com/package/vue-property-decorator" target="_blank">NPM package</a>), these properties will be marked with <span class="code">@Prop()</span> decorator (annotation).</li><li>Add <a href="https://vuejs.org/v2/guide/reactivity.html" target="_blank">"reactive"</a> properties. In our case we'll have single reactive property - current value of the text box.</li><li>Add text box's <span class="code">onchange</span> handler that will be used to bubble the value up to the "builder" and to the web part.</li><li>Few computed properties to be used in the template: <span class="code">styles</span> to reference compiled CSS classes names and <span class="code">inputId</span> for unique text box id.</li></ul>The code: 
<div markdown="1">
{% highlight typescript %}

<script lang="ts">
import { Vue, Component, Prop, Provide } from 'vue-property-decorator';

import {
    IPropertyFieldImageUrlHostProps
} from './IPropertyFieldImageUrlHost';

/**
 * Class-component
 */
@Component
export default class PropertyFieldImageUrlHost extends Vue implements IPropertyFieldImageUrlHostProps {
    @Prop()
    public label: string;

    @Prop()
    public value?: string;

    @Prop()
    public uniqueKey: string;
    
    @Prop()
    public onValueChanged: (value: string) => void;

    /**
     * reactive properties of the component
     */
    public data(): any {
        return {
            // text box value
            inputValue: this.value
        };
    }

    /**
     * Unique input id
     */
    public get inputId(): string {
        return `Input${this.uniqueKey}`;
    }

    /**
     * input onchange event handler
     * @param event 
     */
    private _onChange(event) {
        if (this.onValueChanged) {
            this.onValueChanged(this.$data.inputValue);
        }
    }
}
</script>

{% endhighlight %}
</div>
<h3>PropertyFieldImageUrlHost.vue - &lt;template&gt; Section</h3>This section provides markup of the component with Vue.js bindings to the class-component. The markup here duplicates markup hierarchy of <span class="code">TextField</span> <a href="https://developer.microsoft.com/en-us/fabric#/components/textfield" target="_blank">Office UI Fabric component</a>. 
<div markdown="1">
{% highlight html %}

<template>
    <div :class="$style.ImageUrl">
        <label :for="inputId" class="ms-Label" :class="$style.label">{{ "{{" }}label}}>/label>
        <div class="ms-TextField-fieldGroup" :class="$style.inputWrapper">
            <input type="text" v-model="inputValue" :id="inputId" v-on:change="_onChange" />
        </div>
    </div>
</template>

{% endhighlight %}
</div>
Few things to mention here. <ul><li>All attributes starting with <span class="code">:</span> or <span class="code">v-</span> are special Vue.js attributes needed for data binding.</li><li>Same statement as above could be applied to values wrapped with mustache syntax <span class="code">{{ "{{" }}...}}</span> - similarly to <a href="https://handlebarsjs.com/" target="_blank">handlebars.js</a></li><li><span class="code">v-model</span> attribute is used for <a href="https://vuejs.org/v2/guide/forms.html" target="_blank">form inputs two-way binding</a>.<br />The statement 
<div markdown="1">
{% highlight html %}

<input v-model="searchText">

{% endhighlight %}
</div>
is equal to 
<div markdown="1">
{% highlight html %}

<input
  v-bind:value="searchText"
  v-on:input="searchText = $event.target.value"
>

{% endhighlight %}
</div>
In our situation we could use <span class="code">v-bind:value</span> only and use <span class="code">v-on:input</span> instead of <span class="code">v-on:change</span> because we bubble the input's value up to the web part and call re-render with passing this new value back to the component.<br />But I decided to showcase the usage of <span class="code">v-model</span> attribute. And also it assures that even if the value hasn't been passed back, we'll render correct text in the input. </ul><h3>PropertyFieldImageUrl.ts</h3>Now we're ready to implement the last piece of the control: "builder" class.<br />First, let's import <span class="code">Vue</span> and <span class="code">ImageUrlComponent</span>: 
<div markdown="1">
{% highlight typescript %}

// Importing Vue.js
import Vue from 'vue';
// Improting Vue.js SFC
import ImageUrlComponent from './PropertyFieldImageUrlHost.vue';

{% endhighlight %}
</div>
Now we can implement the code of the "builder" class.<br />Most of the code is standard: we need to implement <span class="code">IPropertyPaneField</span> interface, add some control-specific fields (in our case - <span class="code">value</span>), correctly initialize the instance in the constructor, add <span class="code">onRender</span> handler that is called to render the control: 
<div markdown="1">
{% highlight typescript %}

class PropertyFieldImageUrlBuilder implements IPropertyPaneField<IPropertyFieldImageUrlProps> {
    //Properties defined by IPropertyPaneField
    public type: PropertyPaneFieldType = PropertyPaneFieldType.Custom;
    public targetProperty: string;
    public properties: IPropertyFieldImageUrlPropsInternal;
    private elem: HTMLElement;
    private value: string;
    private changeCB?: (targetProperty?: string, newValue?: any) =&gt void;
    
    public constructor(_targetProperty: string, _properties: IPropertyFieldImageUrlProps) {
        this.targetProperty = _targetProperty;
        this.properties = {
            key: _properties.key,
            label: _properties.label,
            onPropertyChange: _properties.onPropertyChange,
            value: _properties.value,
            onRender: this.onRender.bind(this),
            properties: _properties.properties
        };
        
        this.value = _properties.value;
        if (this.value === undefined) {
            this.value = '';
        }
    }
    
    public render(): void {
        // TODO: render
    }
}

{% endhighlight %}
</div>
Next thing is to handle changes of our component.<br />For handling the changes we just need to implement a handler method an pass it as <span class="code">onValueChanged</span> a property to our control. The handler method should compare new value with the previous one, update web part's property and fire callbacks to bubble new value to the web part. The code of the handler is also pretty standard: 
<div markdown="1">
{% highlight typescript %}

private _onInputChange(newValue: string): void {
    if (this.properties.onPropertyChange && newValue !== this.value) {
        this.properties.onPropertyChange(this.targetProperty, this.value, newValue);
        this.value = newValue;
        this.properties.properties[this.targetProperty] = newValue;
        if (typeof this.changeCB !== 'undefined' && this.changeCB !== null) {
            this.changeCB(this.targetProperty, newValue);
        }
    }
}

{% endhighlight %}
</div>
The last part is rendering.<br />The code for rendering is similar to one described in <a href="http://blog.aterentiev.com/2018/05/using-vuejs-in-sharepoint-framework.html" target="_blank">Default SPFx web part using Vue.js</a>: we need to create <span class="code">Vue</span> component with providing <span class="code">render</span> method to the constructor. This method creates our <span class="code">ImageUrlComponent</span> and passes all needed properties.<br />The difference with the process described in <a href="http://blog.aterentiev.com/2018/05/using-vuejs-in-sharepoint-framework.html" target="_blank">Default SPFx web part using Vue.js</a> is that we're not providing <span class="code">el</span> property directly in <span class="code">Vue</span> constructor. We're calling <span class="code">$mount</span> method instead.<br />This is done because parent container for the control might not be added to the actual DOM at the moment when we're creating the instance of <span class="code">Vue</span>. It can exist in Virtual DOM only. In that case Vue.js will throw an error that the element (parent node) is not found. <span class="code">$mount</span> method allows to use "deferred" mounting.<br />The full code of the file, including described <span class="code">onRender</span> method is listed below. 
<div markdown="1">
{% highlight typescript %}

import { IPropertyPaneField, PropertyPaneFieldType } from '@microsoft/sp-webpart-base';
import { IPropertyFieldImageUrlProps, IPropertyFieldImageUrlPropsInternal } from './IPropertyFieldImageUrl';

// Importing Vue.js
import Vue from 'vue';
// Improting Vue.js SFC
import ImageUrlComponent from './PropertyFieldImageUrlHost.vue';

class PropertyFieldImageUrlBuilder implements IPropertyPaneField<IPropertyFieldImageUrlProps> {
    //Properties defined by IPropertyPaneField
    public type: PropertyPaneFieldType = PropertyPaneFieldType.Custom;
    public targetProperty: string;
    public properties: IPropertyFieldImageUrlPropsInternal;
    private elem: HTMLElement;
    private value: string;
    private changeCB?: (targetProperty?: string, newValue?: any) => void;
    
    public constructor(_targetProperty: string, _properties: IPropertyFieldImageUrlProps) {
        this.targetProperty = _targetProperty;
        this.properties = {
            key: _properties.key,
            label: _properties.label,
            onPropertyChange: _properties.onPropertyChange,
            value: _properties.value,
            onRender: this.onRender.bind(this),
            properties: _properties.properties
        };
        
        this.value = _properties.value;
        if (this.value === undefined) {
            this.value = '';
        }
    }
    
    public render(): void {
        if (!this.elem) {
            return;
        }

        this.onRender(this.elem);
    }

    private onRender(elem: HTMLElement, ctx?: any, changeCallback?: (targetProperty?: string, newValue?: any) => void): void {
        if (!this.elem) {
            this.elem = elem;
        }
        this.changeCB = changeCallback;
        
        const id: string = `ppf-${this.properties.key}`;

        elem.innerHTML = '';

        // root div element of the control
        const element: HTMLDivElement = document.createElement('div');
        element.id = id;
        elem.appendChild(element);

        let vueEl = new Vue({
            render: h => h(ImageUrlComponent, {
                props: {
                    uniqueKey: this.properties.key,
                    value: this.value,
                    label: this.properties.label,
                    onValueChanged: this._onInputChange.bind(this)
                }
            })
        });

        vueEl.$mount(element);
    }

    private _onInputChange(newValue: string): void {
        if (this.properties.onPropertyChange && newValue !== this.value) {
            this.properties.onPropertyChange(this.targetProperty, this.value, newValue);
            this.value = newValue;
            this.properties.properties[this.targetProperty] = newValue;
            if (typeof this.changeCB !== 'undefined' && this.changeCB !== null) {
                this.changeCB(this.targetProperty, newValue);
            }
        }
    }
}

export function PropertyFieldImageUrl(targetProperty: string, properties: IPropertyFieldImageUrlProps): IPropertyPaneField<IPropertyFieldImageUrlProps> {
 return new PropertyFieldImageUrlBuilder(targetProperty, properties);
}

{% endhighlight %}
</div>
<h2>Use the Control in the Web Part</h2>Now we can use the control in the web part similarly to any other Property Pane control.<br />First, we need to import <span class="code">PropertyFieldImageUrl</span> function from <span class="code">PropertyFieldImageUrl.ts</span>. 
<div markdown="1">
{% highlight typescript %}

import { PropertyFieldImageUrl } from '../../propertyFields/imageUrl/PropertyFieldImageUrl';

{% endhighlight %}
</div>
And use the function inside <span class="code">getPropertyPaneConfiguration</span>: 
<div markdown="1">
{% highlight typescript %}

PropertyFieldImageUrl('imageUrl', {
    key: 'imageUrl',
    label: strings.ImageUrlFieldLabel,
    value: this.properties.imageUrl,
    properties: this.properties,
    onPropertyChange: this.onPropertyPaneFieldChanged
})

{% endhighlight %}
</div>
To make this code work you should also define <span class="code">imageUrl</span> property in web part properties interface and <span class="code">ImageUrlFieldLabel</span> in web part's strings.<br />Now the value from the control can be used somewhere in the web part.<br />The full code for this sample can be found <a href="https://github.com/AJIXuMuK/vuejs/tree/master/proppane-control" target="_blank">here</a>.<br />The web part from this demo just shows the value from the control like that:<br /><a href="https://2.bp.blogspot.com/-W1g5ejpWAM4/W5H_JGBKZwI/AAAAAAAAA4Y/euMr9XhpgjUCNWa652dPkAr_b4hQNnlYwCLcBGAs/s1600/Screen%2BShot%2B2018-09-06%2Bat%2B9.08.43%2BPM.png" imageanchor="1" ><img border="0" src="https://2.bp.blogspot.com/-W1g5ejpWAM4/W5H_JGBKZwI/AAAAAAAAA4Y/euMr9XhpgjUCNWa652dPkAr_b4hQNnlYwCLcBGAs/s1600/Screen%2BShot%2B2018-09-06%2Bat%2B9.08.43%2BPM.png" data-original-width="1600" data-original-height="791" width="600" /></a><br/><h2>Yeoman Generator for Property Pane Controls</h2>Similarly to VueSpfx generator, it would be great to have some "add-on" Yeoman generator that creates all the structure for Property Pane Custom Control and provides some sample code.<br />And there is such generator! <a href="https://www.npmjs.com/package/generator-spfx-proppane-control" target="_blank">generator-spfx-proppane-control</a>You can install it using 
<div markdown="1">
{% highlight console %}

npm i -g generator-spfx-proppane-control

{% endhighlight %}
</div>
And use inside you SPFx web part project to create as many Property Pane controls as you like. <b>Note: this generator doesn't work as a standalone generator. You should use it for SPFx projects that have been previously scaffolded.</b><br />The generator supports not only Vue.js but React and No Framework options as well.<br />The documentation can be found <a href="https://github.com/AJIXuMuK/generator-spfx-proppane-control/wiki" target="_blank">here</a>. It's pretty simple but covers main features.<br />Please, leave your feedback or contribute to the generator!<br />It's highly appreciated! <br /><br />And that's it for today!<br />Have fun!