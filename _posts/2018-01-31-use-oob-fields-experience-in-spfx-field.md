---
layout: post
title: Use OOB Fields Experience in SPFx Field Customizer
date: '2018-01-31T16:45:00.000-08:00'
author: Alex Terentiev
tags:
- SharePoint Online
- Field Customizer
- conditional formatting
- O365
- SharePoint Framework
- Office 365
- SharePoint Framework Extensions
- Modern UI
- SharePoint
modified_time: '2018-01-31T16:50:13.753-08:00'
thumbnail: https://3.bp.blogspot.com/-xCe2xIig3u4/WnJdSi0k-XI/AAAAAAAAAt8/0uxkV78df3QBNUnUeivljCpApksl-zLtgCLcBGAs/s72-c/Screen%2BShot%2B2018-01-31%2Bat%2B4.20.03%2BPM.png
blogger_id: tag:blogger.com,1999:blog-3066084330774405472.post-773979052825864266
blogger_orig_url: http://blog.aterentiev.com/2018/01/use-oob-fields-experience-in-spfx-field.html
---

I'm happy to announce that <a href="https://www.npmjs.com/package/@pnp/spfx-controls-react" target="_blank">SharePoint Framework React Controls</a> package starting v1.2.0 includes <b>Field Controls</b> - a set of React components that can be used in Field Customizer to render field's content similarly to OOB experience.<br />Main features of the controls: <ul><li>OOB fields experience for different types of fields (Text, Title, Urls, Lookups, etc.)</li><li>"Factory" helper to select proper Field Control based on field's type</li><li>Ability to provide custom CSS classes and styles to the Field Control</li><li>It's open source</li></ul><a name='more'></a><h2>Why you may need that</h2>The most common scenario to use these controls is when a developer needs to slightly modify oob field rendering, for example, change background or font color or add KPI icon based on some rule.<br />  As Microsoft doesn't provide any API to call fields' default renderer (at least now) a developer needs to create all the oob markup by himself. And some of the fields are pretty complicated. For example, Person or Group provides a "hoverable" list of users selected with Persona Hover Card opened on hover:<br /><a href="https://3.bp.blogspot.com/-xCe2xIig3u4/WnJdSi0k-XI/AAAAAAAAAt8/0uxkV78df3QBNUnUeivljCpApksl-zLtgCLcBGAs/s1600/Screen%2BShot%2B2018-01-31%2Bat%2B4.20.03%2BPM.png" imageanchor="1" ><img border="0" src="https://3.bp.blogspot.com/-xCe2xIig3u4/WnJdSi0k-XI/AAAAAAAAAt8/0uxkV78df3QBNUnUeivljCpApksl-zLtgCLcBGAs/s320/Screen%2BShot%2B2018-01-31%2Bat%2B4.20.03%2BPM.png" width="212" height="320" data-original-width="646" data-original-height="974" /></a><br/>Field Controls provide similar experience and the only thing a developer needs is to provide additional CSS. Here is a Persona Hover Card from <span class="code">FieldUserRenderer</span> control:<br /><a href="https://2.bp.blogspot.com/-Sncxb2EdI4Y/WnJd4qXPfNI/AAAAAAAAAuA/yIXugwGYNkQ0zA30-1qlevZzjLovvpxOQCLcBGAs/s1600/Screen%2BShot%2B2018-01-31%2Bat%2B4.22.40%2BPM.png" imageanchor="1" ><img border="0" src="https://2.bp.blogspot.com/-Sncxb2EdI4Y/WnJd4qXPfNI/AAAAAAAAAuA/yIXugwGYNkQ0zA30-1qlevZzjLovvpxOQCLcBGAs/s320/Screen%2BShot%2B2018-01-31%2Bat%2B4.22.40%2BPM.png" width="320" height="297" data-original-width="620" data-original-height="576" /></a><br /><br />Another scenario is related to lists/document libraries that have multiple views. And potentially some fields should have custom markup in some viees but default one in others. Field Customizer is applied to the <b>field</b>, not to the field in the view. It means that the customization is available in any view in the list.<br />And here developers can use Field Controls as well and implement the condition to use custom logic or Field Controls logic based on view id.<br />Here are links to SharePoint UserVoice requests to provide oob fields rendering. Please, vote for them as well.<br /><a href="https://sharepoint.uservoice.com/forums/329220-sharepoint-dev-platform/suggestions/18810637-access-to-re-use-modern-field-render-controls" target="_blank">https://sharepoint.uservoice.com/forums/329220-sharepoint-dev-platform/suggestions/18810637-access-to-re-use-modern-field-render-controls</a><br /><a href="https://sharepoint.uservoice.com/forums/329220-sharepoint-dev-platform/suggestions/31530607-field-customizer-ability-to-call-ootb-render-meth" target="_blank">https://sharepoint.uservoice.com/forums/329220-sharepoint-dev-platform/suggestions/31530607-field-customizer-ability-to-call-ootb-render-meth</a><h2>Usage</h2><h3>Package Installation</h3>To get started you have to install the following dependency to your project: <span class="code">@pnp/spfx-controls-react</span><br />Enter the following command to install the dependency to your project: 
<div markdown="1">
{% highlight console %}

npm install @pnp/spfx-controls-react --save --save-exact

{% endhighlight %}
</div>
<h3>Package Configuration</h3>Once the package is installed, you will have to configure the resource file of the property controls to be used in your project. You can do this by opening the <span class="code">config/config.json</span> and adding the following line to the <span class="code">localizedResources</span> property: 
<div markdown="1">
{% highlight javascript %}

"ControlStrings": "./node_modules/@pnp/spfx-controls-react/lib/loc/{locale}.js"

{% endhighlight %}
</div>
<h3>Working with Field Controls</h3>The main scenario to use this package is to import <span class="code">FieldRendererHelper</span> class from <span class="code">@pnp/spfx-controls-react/lib/Utilities</span> and to call <span class="code">getFieldRenderer</span> method. This method returns a <span class="code">Promise</span> with a proper field renderer (<span class="code">Promise&lt;JSX.Element&gt;</span>) based on field's type. It means that it will automatically select proper component that should be rendered in this or that field. This method also contains logic to correctly process field's value and get correct text to display (for example, Friendly Text for DateTime fields). As the method returns <span class="code">Promise</span> it should be called in one of React component lifecycle methods, for example, <span class="code">componentWillMount</span> that will occur before <span class="code">render</span>. The resulting field renderer could be saved in the element's state and used later in <span class="code">render</span> method.<br />Here is an example on how it can be used inside custom Field Customizer component (.tsx file): 
<div markdown="1">
{% highlight typescript %}

import { FieldRendererHelper } from '@pnp/spfx-controls-react/lib/Utilities';

//...

export interface IOotbFieldsState {
  fieldRenderer?: JSX.Element;
}

//...

@override
  public componentWillMount() {
    FieldRendererHelper.getFieldRenderer(this.props.value, {
      className: this.props.className,
      cssProps: this.props.cssProps
    }, this.props.listItem, this.props.context).then(fieldRenderer => {
      this.setState({
        fieldRenderer: fieldRenderer
      });
    });
  }

public render(): React.ReactElement<{}> {
    return (
      <div className={styles.cell}>
        {this.state.fieldRenderer}
      </div>
    );
  }

{% endhighlight %}
</div>
<br /><br />And that's it!<br />Here are the links to <a href="https://github.com/SharePoint/sp-dev-fx-controls-react" target="_blank">GitHub Repo</a> and <a href="https://sharepoint.github.io/sp-dev-fx-controls-react/" target="_blank">Package documentation</a>.<br />Please, feel free to post the issues, questions and PRs to the repo.<br />Have fun!