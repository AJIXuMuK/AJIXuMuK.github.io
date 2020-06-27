---
layout: post
title: React Templates Using SPFx Library Components. Part I. Basics.
date: '2019-06-17T08:00:00.000-07:00'
author: Alex Terentiev
tags:
- SharePoint Online
- React Templates
- Template
- SharePoint Framework
- Office 365
- SharePoint Development
- SharePoint
- SPFx Library Component
- SPFx
- React
- O365
- SharePoint Framework Library Component
modified_time: '2019-06-26T12:48:33.687-07:00'
thumbnail: https://1.bp.blogspot.com/-dZYgN04-mhU/XQxk6O0xjUI/AAAAAAAABJg/LwUCUhMGiZArLXXQ4mpFvZt95sGU3UoYgCLcBGAs/s72-c/default-ui.png
blogger_id: tag:blogger.com,1999:blog-3066084330774405472.post-486475278521774157
blogger_orig_url: http://blog.aterentiev.com/2019/06/react-templates-using-spfx-library.html
---

Templating is a pretty powerful approach to provide extensibility to a project/component/library. It can be used to deliver different behaviors and/or look and feel to different customers, or provide extendable open-source libraries.<br />There are a lot of template libraries out there that can be used in your project. One of the most popular is <a href="https://handlebarsjs.com/" target="_blank">Handlebars.js</a>. The problem with these libraries is that in most cases they provide you an ability to define "static" content (basically, HTML and CSS). And if you want to include some actionable content (e.g. handle events from outside the template, etc.) - it might be either tricky or impossible.<br />In the next several blog posts I want to showcase how to create fully functional React templates (with props, events handling etc.) and dynamically load them using new SharePoint Framework project type - <a href="https://docs.microsoft.com/en-us/sharepoint/dev/spfx/library-component-tutorial" target="_blank">Library Component</a>.<br />Technically, this approach can be exposed to non-SPFx projects as well, but it is not the purpose of this blog posts series.<br />If you don't want to read the posts - the code is available <a href="https://github.com/AJIXuMuK/SPFx/tree/master/react-templates" target="_blank">here</a>. Feel free to use it in any way you want. <br /><a name='more'></a><br /><a href="http://blog.aterentiev.com/2019/06/react-templates-using-spfx-library_25.html" target="_blank">Second post - Implementation</a><br /><br /><h2>Objective</h2>So, as mentioned above, templating allows us to provide extensibility to existing projects and implement different behaviors and look and feel for different customers <b>separately</b> of the core component.<br />In these posts we'll be implementing a Tasks List web part that shows a list of tasks and selected task details.<br />We want the web part to have some default UI and also allow to provide custom templates for the components. <br />Default UI will look like that:<br /><a href="https://1.bp.blogspot.com/-dZYgN04-mhU/XQxk6O0xjUI/AAAAAAAABJg/LwUCUhMGiZArLXXQ4mpFvZt95sGU3UoYgCLcBGAs/s1600/default-ui.png" imageanchor="1" ><img border="0" src="https://1.bp.blogspot.com/-dZYgN04-mhU/XQxk6O0xjUI/AAAAAAAABJg/LwUCUhMGiZArLXXQ4mpFvZt95sGU3UoYgCLcBGAs/s1600/default-ui.png" data-original-width="1600" data-original-height="889" width="600" /></a><br />And alternative UI:<br /><a href="https://4.bp.blogspot.com/-C0I5zQcoRiw/XQxlIFDQZzI/AAAAAAAABJk/sWxFdAX9d_kx2OCBUVaJEom9MVkhPFN1QCLcBGAs/s1600/library-component.png" imageanchor="1" ><img border="0" src="https://4.bp.blogspot.com/-C0I5zQcoRiw/XQxlIFDQZzI/AAAAAAAABJk/sWxFdAX9d_kx2OCBUVaJEom9MVkhPFN1QCLcBGAs/s1600/library-component.png" data-original-width="1454" data-original-height="498" width="600" /></a><br /><h2>Initial Implementation Details</h2>From components perspective we'll have <ol><li>Task - to display a single task item in the list</li><li>Tasks List - to display a collection of tasks</li><li>Task Details - to display details of the selected task</li></ol>Default rendering of these component will be included in the web part's project, but we'll also provide an ability to dynamically connect to SPFx Library Component and get these components from there.<br />As we're going to have different implementations of the same components in different projects/libraries we also need to define "contracts" - interfaces and naming conventions that must be used in all implementations. It will allow to dynamically load elements and "know" what classes/functions/etc. we can use.<br />Here are the common interfaces and types we need: <ul><li><span class="code">TemplateType</span> - type to define all possible templates: <span class="code">task-list</span>, <span class="code">task</span>, <span class="code">task-details</span></li><li><span class="code">ITask</span> - interface to define Task properties</li><li><span class="code">ITaskListProps</span> - <span class="code">TaskList</span> React component's props</li><li><span class="code">ITaskProps</span> - <span class="code">Task</span> React component's props</li><li><span class="code">ITaskDetailsProps</span> - <span class="code">TaskDetails</span> React component's props</li><li><span class="code">ITemplateFactory</span> - Templates Factory interface, main entry point for templates libraries.  </ul>Below is the implementation of the contracts. In my case I just created a separate TypeScript file - <span class="code">CommonTypes.ts</span> that can be copied to any project. There are other possible approaches here like type definition file (d.ts) or creating a separate module and linking it to each library. 
<div markdown="1">
{% highlight typescript %}

import * as React from 'react';

/**
 * Available types of templates
 */
export type TemplateType = 'task-list' | 'task' | 'task-details';

/**
 * Task statuses
 */
export enum TaskStatus {
  NotStarted,
  InProgress,
  Resolved,
  Closed
}

/**
 * Task interface
 */
export interface ITask {
  id: string;
  title: string;
  description: string;
  status: TaskStatus;
  estimate: number;
  spentTime: number;
  dueDate: Date;
  assignedTo?: string;
}

/**
 * Props for TaskList component
 */
export interface ITaskListProps {
  /**
   * Tasks to display
   */
  tasks: ITask[];
  /**
   * React component (template) to use to display tasks
   */
  taskTemplate: React.ComponentClass<ITaskProps>;
  /**
   * Task selected handler
   */
  onTaskSelected: (task: ITask) => void;
  /**
   * Localized strings
   */
  strings: any;
}

/**
 * Props for Task component
 */
export interface ITaskProps extends ITask {
  /**
   * Selected handler
   */
  onSelected: () => void;
  /**
   * Localized strings
   */
  strings: any;
}

/**
 * Props for TaskDetails component
 */
export interface ITaskDetailsProps extends ITask {
  /**
   * Localized strings
   */
  strings: any;
}

/**
 * Main entry point of the template library - Template factory
 */
export interface ITemplateFactory {
  /**
   * Get the template component (React component) based on TemplateType
   */
  getTemplateComponent: (templateType: TemplateType) => React.ComponentClass<any> | null;
}

{% endhighlight %}
</div>
<br /><h2>React Dynamic Components</h2>React by its nature allows to use dynamic components names right in JSX and TSX files - any tag name starting with Capital letter will compile in <span class="code">createComponent</span> method call. So, you can have something like: 
<div markdown="1">
{% highlight typescript %}

public render(): React.ReactElement<IProps> {
  const MyComponent = components[componentName];
  return <MyComponent {...someProps} />;
}

{% endhighlight %}
</div>
And it will be compiled into: 
<div markdown="1">
{% highlight typescript %}

public render(): React.ReactElement<IProps> {
  const MyComponent = components[componentName];
  return React.createElement(MyComponent, someProps);
}

{% endhighlight %}
</div>
Keep in mind the value used as a tag name must be a component or class reference, not just a string. That's why we defined return type of Factory method as <span class="code">React.ComponentClass&lt;any&gt; | null</span>.<br />In our case we can use React dynamic components (or dynamic tag names) and <span class="code">ITemplateFactory</span> interface to render needed component with dynamic template: 
<div markdown="1">
{% highlight typescript %}

public render(): React.ReactElement<IProps> {
  const TaskList = this.props.templateFactory.getTemplateComponent('task-list') as React.ComponentClass<ITaskListProps>;
  return <TaskList {...someProps} />;
}

{% endhighlight %}
</div>
<br /><h2>SharePoint Framework Library Components</h2>The last but not least part of the puzzle is <a href="https://docs.microsoft.com/en-us/sharepoint/dev/spfx/library-component-overview" target="_blank">SPFx Library Components</a>.<br />From the official documentation: <span style="text-decoration: italic;">Library component type enables you to have independently versioned and deployed code, which is served automatically for the SharePoint Framework components with a deployment through app catalog. Library component provides you alternative option to create shared code, which can be then used and referenced cross all the components in the tenant.</span> You can think of Library Components as DLLs - separate packages that can be included (statically or dynamically) to other projects. And that is a great fit for our needs.<br /><br /><b>Note: current latest version of SharePoint Framework is 1.8.2. Library Components feature is in preview and is subject to change. It is not currently supported for use in production environments.</b><br /><br />So, we can (and will) use Library Component to implement alternative rendering for our tasks list and task details, and dynamically load that library by the web part. <br /><h2>Next Steps</h2>The <a href="http://blog.aterentiev.com/2019/06/react-templates-using-spfx-library_25.html">next post</a> walks you through the whole implementation, including Templates library loader, default Template Factory, Library Component with alternative templates, and dynamic rendering of tasks list and task details. <br /><br />That's all for today!<br />Have fun! 