---
layout: post
title: React Templates Using SPFx Library Components. Part II. Implementation.
date: '2019-06-25T05:00:00.000-07:00'
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
modified_time: '2019-09-10T16:38:00.971-07:00'
blogger_id: tag:blogger.com,1999:blog-3066084330774405472.post-6044219796498671200
blogger_orig_url: http://blog.aterentiev.com/2019/06/react-templates-using-spfx-library_25.html
---

In the <a href="http://blog.aterentiev.com/2019/06/react-templates-using-spfx-library.html" target="_blank">first post</a> we've already discussed the objective, as well as basics, or strategy, of how to create fully functional React templates (with props, events handling etc.) and dynamically load them using new SharePoint Framework project type - <a href="https://docs.microsoft.com/en-us/sharepoint/dev/spfx/library-component-tutorial" target="_blank">Library Component</a>.<br />This post covers implementation details: main web part, dynamic template factory loading, and library component with alternative UI.<br />If you don't want to read the post - the code is available <a href="https://github.com/AJIXuMuK/SPFx/tree/master/react-templates" target="_blank">here</a>. Feel free to use it in any way you want. <br /><a name='more'></a><br /><a href="http://blog.aterentiev.com/2019/06/react-templates-using-spfx-library.html" target="_blank">Previous post - Basics</a><br /><br /><h2>Web Part</h2>As mentioned in the previous post, we'll be creating 2 different UIs for Tasks list.<br />The default one, implemented as part of client side web part, will look like that:<br /><img border="0" src="{{site.baseurl}}/assets/images/posts/2019/default-ui.png" /><br />And alternative one, implemented as a separate Library Component:<br /><img border="0" src="{{site.baseurl}}/assets/images/posts/2019/library-component.png" /><br /><br /><h3>Initial Preparation</h3>And let's start with the web part implementation.<br />As a template we'll use Web Part with React Framework.<br /><img border="0" src="{{site.baseurl}}/assets/images/posts/2019/scaffold.png" /><br /><br /><h3>Contracts</h3>As mentioned in the first post, there are "contracts" that must be used in all implementations of components. And in this solution all of them are defined in <span class="code">CommonTypes.ts</span> file that should be copied in the projects. So, let's create <span class="code">common</span> folder in web part's code directory and copy the file there.<br />Now we have interfaces for templates factory, as well as for all dynamic React components: Tasks List, Task, and Task Details.<br /><br /><h3>Web Part's Main Component</h3>Now let's modify web part's main component (in this example - <span class="code">TasksTemplates</span>) to render what we really need instead of default "Learn more" UI.<br />At this point of time we know that there is some <span class="code">ITemplateFactory</span> interface with <span class="code">getTemplateComponent</span> method. We don't know (yet) how to instantiate it, but we can use the interface definition to dynamically get needed templates and render them in <span class="code">TasksTemplates</span> component.<br />So, first, let's update <span class="code">ITasksTemplatesProps</span> to expect templates factory as well as list of tasks to be rendered: <br />
<div markdown="1">
{% highlight typescript %}
import { ITemplateFactory, ITask } from '../../common/CommonTypes';

/**
 * Main web part component props
 */
export interface ITasksTemplatesProps {
  /**
   * Selected template factory
   */
  templateFactory: ITemplateFactory;
  /**
   * Tasks to render
   */
  tasks: ITask[];
}

{% endhighlight %}
</div>
We also need to implement component's state to store selected task: <br />
<div markdown="1">
{% highlight typescript %}
export interface ITaskTemplatesState {
  selectedTask?: ITask;
}

{% endhighlight %}
</div>
Now we can modify <span class="code">render</span> method to get templates from the factory and render them inside the component. <br />
<div markdown="1">
{% highlight typescript %}
public render(): React.ReactElement<ITasksTemplatesProps> {
    const {
      tasks,
      templateFactory
    } = this.props;

    const {
      selectedTask
    } = this.state;

    //
    // Any capitalized variable can be used as a React component in .tsx (.jsx) files.
    // We're getting the templates (React components) from template factory to use them in the markup
    //
    const TaskList = templateFactory.getTemplateComponent('task-list') as React.ComponentClass<ITaskListProps>;
    const Task = templateFactory.getTemplateComponent('task') as React.ComponentClass<ITaskProps>;
    const TaskDetails = templateFactory.getTemplateComponent('task-details') as React.ComponentClass<ITaskDetailsProps>;

    return (
      <div className={styles.tasksTemplates}>
        <TaskList strings={strings} tasks={tasks} taskTemplate={Task} onTaskSelected={task => {
          this.setState({
            selectedTask: task
          });
        }} />
        {selectedTask &amp;&amp; <TaskDetails {...selectedTask} strings={strings} />}
      </div>
    );
  }

{% endhighlight %}
</div>
<br />Two additional interesting things to mention.<br />First, when getting templates from the factory we're casting them to <span class="code">React.ComponentClass&lt;IProps&gt;</span>. It allows us to use TypeScript's type checking for dynamic component's props.<br />Second, <span class="code">TaskList</span> component has a property <span class="code">taskTemplate: React.ComponentClass&lt;ITaskProps&gt;</span>. And using such a property we can pass one template to another. So, if needed, we can even combine templates from different sources and pass them one to another.<br /><br /><h3>TaskList, Task, TaskDetails - Default Implementation</h3>Next step is to implement default templates for <span class="code">TaskList</span>,<span class="code">Task</span> and <span class="code">TaskDetails</span> components.<br />There is nothing interesting in <span class="code">Task</span> and <span class="code">TaskDetails</span> implementations. You can find them in <a href="https://github.com/AJIXuMuK/SPFx/tree/master/react-templates" target="_blank">GitHub repo</a> for the solution.<br />But let's look at <span class="code">TaskList</span> implementation. As shown above, one of the properties for the list is <span class="code">taskTempalte</span> that contains a component to render each task in the list.<br />To use it we need to apply the same approach as in <span class="code">TasksTemplates</span> component - we'll get the value from props and store it in capitalized variable: <br />
<div markdown="1">
{% highlight typescript %}
public render(): React.ReactElement<ITaskListProps> {
    const {
      tasks,
      taskTemplate,
      onTaskSelected,
      strings
    } = this.props;

    // any capitalized variable can be used and React element
    const TaskTemplate = taskTemplate;

    return (
      <div className={styles.taskList}>
        {tasks.map(t => {
          return <TaskTemplate {...t} strings={strings} onSelected={() =gt; { onTaskSelected(t); }} />;
        })}
      </div>
    );
  }

{% endhighlight %}
</div>
Cool, isn't it? We use one dynamic React component from another!<br />Now, let's implement default templates factory that will work with the components mentioned above: <br />
<div markdown="1">
{% highlight typescript %}
import * as React from 'react';
import { ITemplateFactory, TemplateType } from './common/CommonTypes';
import { Task } from './components/task/Task';
import { TaskDetails } from './components/taskDetails/TaskDetails';
import { TaskList } from './components/taskList/TaskList';

/**
 * Default template factory
 */
export class DefaultTemplateFactory implements ITemplateFactory {
  public getTemplateComponent(templateType: TemplateType): React.ComponentClass<any> | null {
    switch (templateType) {
      case 'task':
        return Task;
      case 'task-details':
        return TaskDetails;
      case 'task-list':
        return TaskList;
    }
  }
}

{% endhighlight %}
</div>
<br /><h2>Library Component with Alternative Templates</h2>Library Component is a new type of SPFx project that is GAd in version 1.9.0. It allows developers to create some "shared" code that is deployed to App Catalog and referenced by other SPFx components (Web Parts, Extensions, and even other Library Components).<br />If you run SPFx Yeoman generator version 1.9.0 or higher, you'll see a new option - Library.<br /><img border="0" src="{{site.baseurl}}/assets/images/posts/2019/scaffold-library.png" /><br />This option will generate new SPFx project with a well-know structure - same as for web parts and extensions.<br /><img border="0" src="{{site.baseurl}}/assets/images/posts/2019/library-structure.png" /><br />The main differences here is not empty <span class="code">index.ts</span> file in the root of <span class="code">src</span> folder. In case of Library Component it actually shows what will be exported from the library. In default implementations it's a class with a single property called <span class="code">name</span> to return library's name: <br />
<div markdown="1">
{% highlight typescript %}
export default class TemplatesLibrary {
  public name(): string {
    return 'MyLibComponentLibrary';
  }
}

{% endhighlight %}
</div>
<b>It's important to mention that while scaffolding a Library Component you need to set <i>Do you want to allow the tenant admin the choice of being able to deploy the solution to all sites immediately without running any feature deployment or adding apps in sites?</i> to <i>Yes</i> if you want it to be available across tenant.</b>Let's modify this class to be a template factory.<br />First, similarly to web part, we need to copy our "contracts" - <span class="code">CommonTypes.ts</span> to the project. And, again, I'll copy it in <span class="code">common</span> folder.<br /><img border="0" src="{{site.baseurl}}/assets/images/posts/2019/common-types.png" /><br />Next - implement all used components - <span class="code">TaskList</span>,<span class="code">Task</span> and <span class="code">TaskDetails</span>. The implementation is pretty standard so no need to post it here. The only thing to mention - by default Library Component project doesn't reference React library. So, for our implementation we'll need to install additional modules: <br />
<div markdown="1">
{% highlight typescript %}
npm i --save --save-exact @types/react@16.4.2 @types/react-dom@16.0.5 react@16.7.0 react-dom@16.7.0 office-ui-fabric-react@5.132.0

{% endhighlight %}
</div>
Now we can implement <span class="code">ITemplateFactory</span> interface in our default export - <span class="code">TemplatesLibrary</span> class: <br />
<div markdown="1">
{% highlight typescript %}
/**
 * Default export from the SPFx Library Component is a template factory
 */
export default class TemplatesLibrary implements ITemplateFactory {
  public getTemplateComponent(templateType: TemplateType): React.ComponentClass<any> | null {
    switch (templateType) {
      case 'task':
        return Task;
      case 'task-details':
        return TaskDetails;
      case 'task-list':
        return TaskList;
    }
  }
}

{% endhighlight %}
</div>
And that's it! We have our alternative UI implementation! Pretty easy, right?<br />Now we can package Library Component solution, deploy it to App Catalog and make it available for other SPFx solutions in the tenant.<br /><br /><h2>Secret Ingredient: SPComponentLoader</h2>Now we have the web part, we have default UI for our Tasks list, we have a separate library with alternative UI. But how will we select the appropriate templates factory and load templates?<br />The answer is simple - <span class="code">SPComponentLoader</span>. (<b>Unfortunately, class definition was removed from the documentation for some reason. But you can find some mentions of it <a href="https://docs.microsoft.com/en-us/sharepoint/dev/spfx/web-parts/guidance/connect-to-sharepoint-using-jsom" target="_blank">here</a>, <a href="https://docs.microsoft.com/en-us/sharepoint/dev/spfx/web-parts/guidance/reference-third-party-css-styles" target="_blank">here</a>, <a href="https://docs.microsoft.com/en-us/sharepoint/dev/spfx/web-parts/basics/add-an-external-library" target="_blank">here</a> and <a href="https://docs.microsoft.com/en-us/sharepoint/dev/spfx/web-parts/get-started/add-jqueryui-accordion-to-web-part" target="_blank">here</a>. There is also a separate NPM package for the loader <a href="https://www.npmjs.com/package/@microsoft/sp-loader" target="_blank">sp-loader</a></b>). We can use this class to either load any JavaScript file, or load an SPFx component by its ID. The latter is exactly what we can use for Library Components as they're still SPFx components with defined ID.<br />For that purposes let's define a <span class="code">componentId</span> property in our web part. If the property is set then we'll load the component by ID using <span class="code">SPComponentLoader</span>, otherwise we'll just use default template factory and default UI. <br />
<div markdown="1">
{% highlight typescript %}
/**
 * web part props
 */
export interface ITasksTemplatesWebPartProps {
  /**
   * The id of Library Component SPFx component
   */
  componentId: string;
}

{% endhighlight %}
</div>
Next, let's implement <span class="code">TemplateFactoryLoader</span> class that will asynchronously load needed template factory. Additionally, it will cache last loaded factory to avoid additional traffic: <br />
<div markdown="1">
{% highlight typescript %}
import { ITemplateFactory } from './common/CommonTypes';
import { SPComponentLoader } from '@microsoft/sp-loader';
import { DefaultTemplateFactory } from './DefaultTemplateFactory';

/**
 * Loader of the TemplateFactory, or, more widely - SPFx Library Component loader
 */
export class TemplateFactoryLoader {
  /**
   * last selected componetId
   */
  private static _lastComponentId: string | undefined;
  /**
   * last loaded template factory
   */
  private static _templateFactory: ITemplateFactory;

  /**
   * Loads SPFx Library Component and instantiates template factory from loaded module
   * @param componentId SPFx Library Component componentId
   */
  public static async loadTemplateFactory(componentId?: string): Promise<ITemplateFactory> {
    // we don't want to load the module more than once
    if (componentId !== this._lastComponentId || !this._templateFactory) {
      this._lastComponentId = componentId;

      // if componentId is not defined we'll use the default template factory
      if (!componentId) {
        this._templateFactory = new DefaultTemplateFactory();
      }
      else {
        // loading the module (component)
        const factoryModule: any = await SPComponentLoader.loadComponentById(componentId);
        // instantiating the template factory
        this._templateFactory = new factoryModule.TemplatesLibrary();
      }
    }

    return this._templateFactory;
  }
}

{% endhighlight %}
</div>
Final step is to request template factory from our web part. Note, that <span class="code">loadTemplateFactory</span> is an asynchronous method. To use it during web part's rendering we'll need to override <span class="code"> isRenderAsync</span> property to return <span class="code">true</span> and call <span class="code">this.renderCompleted();</span> to notify that we're finished with the rendering: <br />
<div markdown="1">
{% highlight typescript %}
/**
 * Notifies SPFx engine that rendering is asynchronous
 */
protected get isRenderAsync(): boolean {
  return true;
}

public async render(): Promise<void> {
  // getting template factory based on selected componentId asynchronously
  const templateFactory = await TemplateFactoryLoader.loadTemplateFactory(this.properties.componentId);

  const element: React.ReactElement<ITasksTemplatesProps> = React.createElement(
    TasksTemplates,
    {
      tasks: this._tasks, // some mock tasks
      templateFactory: templateFactory
    }
  );

  ReactDom.render(element, this.domElement);

  // notifying SPFx engine that rendering has been finished
  this.renderCompleted();
  }

{% endhighlight %}
</div>
Now we can switch the UI by providing a single property in web part's property pane!<br />To test that, run the web part in SharePoint-hosted workbench, copy the id from Component Library's manifest (in this sample it is <span class="code">cd6e18e2-a8f3-4f97-9f07-7dfcecdd47ed</span>) and paste it into Component Id property.<br />Voi-la!<br /><img border="0" src="{{site.baseurl}}/assets/images/posts/2019/task-templates.gif" /><br /><br /><h2>Conclusion</h2>SharePoint Framework gives us incredible and powerful instruments to develop customizations.<br />And Library Component project type is a new step forward to simplify our implementation, code reusing and sharing.<br />Together with <span class="code">SPComponentLoader</span> it allows us to implement React templates with dynamic loading of components. Similarly, it can be used to develop different data adapters, dynamic behaviors, etc.<br />I'm really excited and looking forward to including Library Component in GA version of SPFx.<br /><br />That's all for today!<br />Have fun!