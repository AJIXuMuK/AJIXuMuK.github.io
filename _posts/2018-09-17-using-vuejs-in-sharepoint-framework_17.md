---
layout: post
title: 'Using Vue.js in SharePoint Framework Applications. Part V: Use React Components
  inside Vue.js Solution'
date: '2018-09-17T16:40:00.001-07:00'
author: Alex Terentiev
tags:
- Office UI Fabric
- SharePoint Online
- Vuejs
- SharePoint Framework
- Office 365
- PnP
- SharePoint
- Client Side Web Part
- Client Web Part
- TypeScript
- Vue
- React
- generator-sharepoint
- O365
- Modern UI
modified_time: '2018-09-17T16:42:36.032-07:00'
blogger_id: tag:blogger.com,1999:blog-3066084330774405472.post-1964505514297976569
blogger_orig_url: http://blog.aterentiev.com/2018/09/using-vuejs-in-sharepoint-framework_17.html
---

This is the fifth post about SharePoint Framework and Vue.js. In this post I want to show how to use React components inside Vue.js-based SharePoint Framework solutions.<br /><br />List of posts:<br /><ol><li><a href="http://blog.aterentiev.com/using-vuejs-in-sharepoint-framework" target="_blank">Whats and Whys</a></li><li><a href="http://blog.aterentiev.com/using-vuejs-in-sharepoint-framework" target="_blank">Default SPFx web part using Vue.js</a></li><li><a href="http://blog.aterentiev.com/using-vuejs-in-sharepoint-framework" target="_blank">Yeoman generator with Vue support</a></li><li><a href="http://blog.aterentiev.com/using-vuejs-in-sharepoint-framework" target="_blank">Web Part Property Pane Control</a></li><li>Use React Components inside Vue.js Solution (this post)</li></ol><b>Code: </b><a href="https://github.com/AJIXuMuK/vuejs/tree/master/react-in-vue" target="_blank">https://github.com/AJIXuMuK/vuejs/tree/master/react-in-vue</a>.<br /><a name='more'></a>In previous posts we discussed practically all the steps and tools that you need to develop SharePoint Framework solution using Vue.js framework.<br />But there is still uncovered question that might impact on developer's decision to use Vue.js in SPFx solution.<br />The question is: Can we and How to use React components inside Vue.js projects?<br />Why should we care about React in Vue? Because Microsoft bets on React and there are a lot of reusable components that simplify SPFx development: <ul><li>There is <a href="https://developer.microsoft.com/en-us/fabric" target="_blank">Office UI Fabric</a> framework to create Office 365-like UI. And it provides React component to reuse</li><li>There are community reusable React components for SPFx solutions: for <a href="https://sharepoint.github.io/sp-dev-fx-property-controls/" target="_blank">Property Panes</a> and <a href="https://sharepoint.github.io/sp-dev-fx-controls-react/" target="_blank">Web Parts/Extensions</a></li></ul>Good news everyone! <b>We can use React components inside Vue.js!</b><br />And it's actually pretty easy to do because there is an open-source project <a href="https://github.com/akxcv/vuera" target="_blank">Vuera</a> that allows you to integrate Vue.js and React.<br />And let's see how we can use it in SPF solution.<br /><h2>Initial Project Configuration</h2>First, let's create our project.<br />I'll be using <a href="https://www.npmjs.com/package/generator-vuespfx" target="_blank">VueSPFx</a> Yeoman generator to provision all Vue.js references that we need automatically.<br />And let's use Web Part as a playground as it's much easier to debug and render components in there. 
<div markdown="1">
{% highlight console %}

yo vuepsfx

{% endhighlight %}
</div>
After the project has been created we need to reference React libraries: 
<div markdown="1">
{% highlight console %}

npm i --save --save-exact react@15.6.2 react-dom@15.6.2 @types/react@15.6.6 @types/react-dom@15.5.6

{% endhighlight %}
</div>
I'm using <span class="code">--save-exact</span> to reference exactly the same versions of React libraries that are used by default when provisioning SPFx project using OOTB Yeoman Generator with React Framework<br />Next, let's reference <a href="https://github.com/akxcv/vuera" target="_blank">Vuera</a>: 
<div markdown="1">
{% highlight console %}

npm i --save vuera

{% endhighlight %}
</div>
Additionally, let's reference <a href="https://sharepoint.github.io/sp-dev-fx-controls-react/" target="_blank">PnP Reusable controls</a> to showcase how to use these components inside Vue.js web part. 
<div markdown="1">
{% highlight console %}

npm install @pnp/spfx-controls-react --save --save-exact

{% endhighlight %}
</div>
Now we have all the configurations in place and let's start development. <h2>Office UI Fabric React Button, WebPartTitle from PnP Reusable Controls and Custom React Component in Vue.js Web Part</h2>In the sample project here I want to show you how to referect Office UI Fabric Button, WebPartTitle component from PnP Reusable controls and custom React component inside a Vue.js web part.<br />The final UI of the web part will be as simple as that:<br /><img border="0" src="{{site.baseurl}}/assets/images/posts/2018/react-in-vue.png" /><br />Let's first create a custom React component to reference it in the Vue.js web part.<br />For that let's add <span class="code">ReactComponent</span> folder as a subfolder of <span class="code">components</span> directory and add <span class="code">ReactComponent.tsx</span> file into it:<br /><img border="0" src="{{site.baseurl}}/assets/images/posts/2018/react-component.png" /><br />The content of the file (and of the component) is pretty simple: let's make it stateless with the only property named <span class="code">text</span> in the <span class="code">props</span> object. And let's just render this property inside a <span class="code">div</span>: 
<div markdown="1">
{% highlight typescript %}

import * as React from 'react';

/**
 * Simple component props
 */
export interface IReactComponentProps {
    text: string;
}

/**
 * Sample component with no state
 */
export class ReactComponent extends React.Component<IReactComponentProps, {}> {
    constructor(props: IReactComponentProps) {
        super(props);
    }

    public render(): React.ReactElement<IReactComponentProps> {
        return (<div>{this.props.text}</div>);
    }
}

{% endhighlight %}
</div>
<br />Now let's look at Vuera's documentation to check how to inject React components into Vue.js (<a href="https://github.com/akxcv/vuera#react-in-vue---preferred-usage" target="_blank">Preferred usage</a>): <img border="0" src="{{site.baseurl}}/assets/images/posts/2018/react-in-vue-preferred.png" /><br />As you can see form the screenshot above, Vuera is set up as a Vue.js plugin, and later you can just reference you React component in Vue.js component <span class="code">components</span> property: 
<div markdown="1">
{% highlight typescript %}

components: { 'my-react-component': MyReactComponent }

{% endhighlight %}
</div>
Unfortunately, it doesn't work if you use <a href="https://github.com/vuejs/vue-class-component/tree/master" target="_blank">vue-class-component</a> decorator as it expects <span class="code">components</span> property to be of type: 
<div markdown="1">
{% highlight typescript %}

components?: { [key: string]: Component<any, any, any, any> | AsyncComponent<any, any, any, any> };

{% endhighlight %}
</div>
And React components are neither <span class="code">Component</span> nor <span class="code">AsyncComponent</span> that are both Vue.js custom types.<br />So, we can use another way of injecting React components inro Vue.js that is also mentioned in the documentation (<a href="https://github.com/akxcv/vuera#react-in-vue---without-the-vue-plugin" target="_blank">without plugin</a>):<br /><img border="0" src="{{site.baseurl}}/assets/images/posts/2018/react-in-vue-without-plugin.png" /><br />Both options work perfectly. I would prefer the second one as you can use custom elements' names in the template for each component instead of using generic <span class="code">react</span> tag and providing actual component's name as a bind attribute.<br />Let's apply the "HOC API" approach to our project: 
<div markdown="1">
{% highlight typescript %}

// importing needed components
import { ReactComponent } from '../ReactComponent/ReactComponent';
import { WebPartTitle } from "@pnp/spfx-controls-react/lib/WebPartTitle";
import { PrimaryButton } from 'office-ui-fabric-react/lib/Button';

// importing plugin and wrapper
import { VuePlugin, ReactInVue } from 'vuera';

//
// setting up the plugin to work with React components inside Vue
//
Vue.use(VuePlugin);

/**
 * registering components
 */
@Component({
    components: {
        'react-component': ReactInVue(ReactComponent),
        'web-part-title': ReactInVue(WebPartTitle),
        'primary-button': ReactInVue(PrimaryButton)
    }
})
export default class ReactInVueWebPart extends Vue implements IReactInVueWebPartProps {
// ...

{% endhighlight %}
</div>
Now all the React components can be used in the template (<span class="code">.vue</span> file): 
<div markdown="1">
{% highlight html %}

<web-part-title :title="title" :displayMode="displayMode" @updateProperty="_onTitleUpdate" />
<react-component :text="description" />
<primary-button text="Click Me!" @onClick="_onButtonClicked" />

{% endhighlight %}
</div>
<span class="code">title, displayMode, description</span> are the properties of the Vue.js component and <span class="code">_onTitleUpdate, _onButtonClicked</span> are the methods of the class: 
<div markdown="1">
{% highlight typescript %}

/**
 * Component's properties
 */
export interface IReactInVueWebPartProps {
    description: string;
    title: string;
    displayMode: DisplayMode;
    onButtonClicked: () => void;
    onTitleChanged: (value: string) => void;
}

export default class ReactInVueWebPart extends Vue implements IReactInVueWebPartProps {

    /**
     * implementing ISimpleWebPartProps interface
     */
    @Prop()
    public description: string;
    @Prop()
    public title: string;
    @Prop()
    public displayMode: DisplayMode;
    @Prop()
    public onButtonClicked: () => void;
    @Prop()
    public onTitleChanged: (value: string) => void;
    
    private _onTitleUpdate(value: string): void {
        if (this.onTitleChanged) {
            this.onTitleChanged(value);
        }
    }

    private _onButtonClicked(): void {
        if (this.onButtonClicked) {
            this.onButtonClicked();
        }
    }
}

{% endhighlight %}
</div>
 If you look at the code in the provided repo, there is a commented additional option that can be used:<br />
<div markdown="1">
{% highlight typescript %}

/**
* Vue component from custom component
*/
//@ReactComponentDecorator(ReactComponent, 'react-component')
//public reactComponent: ReactComponent;
/**
* Vue compoenent from SPFx Reusable React controls repo
*/
//@ReactComponentDecorator(WebPartTitle, 'web-part-title')
//public webPartTitle: WebPartTitle;

/**
* Vue compoenent from Office UI Fabric Primary Buttom
*/
//@ReactComponentDecorator(PrimaryButton, 'primary-button')
//public primaryButton: PrimaryButton;

{% endhighlight %}
</div>
This approach uses custom <span class"code">ReactComponentDecorator</span> that is located in the <a href="https://github.com/AJIXuMuK/vuejs/blob/master/react-in-vue/src/utils/ReactComponentDecorator.ts" target="_blank">same repository</a> and allows to mark Vue.js component's class property as "component" and provide its tag and type. Using this custom decorator we can use "recommended" approach from Vuera and no need to use <span class="code">ReactInVue</span> HOC API (Note: it's not necessary to use this decorator, it's just an additional exercise on how to create custom decorators for Vue.js).<br /><br />The only left part here is to update web part's code to pass all the properties to the Vue.js component while creating it in <span class="code">render</span> method: 
<div markdown="1">
{% highlight typescript %}

let el = new Vue({
  el: `#${id}`,
  render: h => h(ReactInVueWebPartComponent, {
    props: {
      description: this.properties.description,
      title: this.properties.title,
      displayMode: this.displayMode,
      onTitleChanged: (newTitle: string) => { this.properties.title = newTitle; },
      onButtonClicked: () => { alert('Button Clicked!'); }
    }
  })
});

{% endhighlight %}
</div>
<h2>Conclusion</h2>As a result of this blog post we've created a Vue.js web part that shows how to use React components inside Vue.js SPFx solutions.<br />It shows how to use all the benefits of Office UI Fabric components, PnP reusable React controls as well as custom developed React components in Vue.js applications.<br />If you're in love with Vue.js but consider to switch to React because it simplifies SPFx development, now you can stick to Vue.js and just reuse React controls in the same manner if you'd be a React developer.<br />And here's a small screen recording of the implemented web part:<br /><img border="0" src="{{site.baseurl}}/assets/images/posts/2018/ReactInVue1.gif" /><br /><br />And that's it for today!<br />Have fun!  