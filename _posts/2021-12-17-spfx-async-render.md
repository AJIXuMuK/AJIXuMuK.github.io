---
layout: post
description: Async rendering in SharePoint Framework Web Parts
title: Async Render in SPFx Web Parts
slug: async-render-spfx-web-parts
featured_image: assets/images/posts/2021/2021-10-07-spfx-async-render.png
featured_image_thumbnail: assets/images/posts/2021/2021-10-07-spfx-async-render.png
tags:
  - Microsoft 365 PnP
  - SharePoint
  - SharePoint Framework
  - SPFx
  - SPFx Web Parts
  - PnP
lastmod: '2021-12-17T17:00:23.679Z'
date: '2021-12-17T17:00:23.679Z'
---
When you develop a web part, you often need to request a data from some data source first, and then display the content for the user. And during the process, you want to display some kind of loading indicator to inform user that the app is working and not stale.

If the data is requested once, and you're not requesting tons of information, probably the best way would be to use `onInit` method of the web part. It allows to ensure the data is loaded before the web part is rendered. And it also provides a shimmer of the loading, which is neat.

But if you need refresh your data based on web part's state and properties - there is another approach you can use.

## Asynhronous `render` in SPFx Web Parts
SharePoint Framework engine allows you to "mark" `render` method as async. This means that you can perform asynchronous operations in the `render` method, and the core will wait for the completion before re-rendering the web part.
To do so, you need to make 2 changes in your code.
### Override `isRenderAsync` property
In your web part class, override `isRenderAsync` property to return `true`:

```typescript
  protected get isRenderAsync(): boolean {
    return true;
  }
```
This will inform SPFx that you're going to have asynchronous operations in the `render` method.

### Modify `render` method
There are 2 things to do in the `render` method:
- As the method is asynchronous, it should return a promise, like `Promise<void>`
- You need to call `this.renderCompleted()` when the rendering is completed.

So, you `render` method structure should look like this:

```typescript
public async render(): Promise<void> {
  // ... your code here ...
  this.renderCompleted();
}
```

## When to use this approach
Good example of when you should use this approach is described in the [documentation](https://docs.microsoft.com/en-us/javascript/api/sp-webpart-base/baseclientsidewebpart?view=sp-typescript-latest#renderCompleted_error_):
> One such example is web parts that render content in an IFrame. The web part initiates the IFrame rendering in the render() API but the actual rendering is complete only after the iframe loading completes.

If you have some really heavy and long-running operations in your web part, I would not recommend to use this approach. In these cases, it's better to implement custom "loading" UI and inform user on the process of the operations (for example, display [progress](https://pnp.github.io/sp-dev-fx-controls-react/controls/Progress/) control).

But for simple scenarios, this approach is really handy and easy to implement.

That's all for today!<br />
Have fun!
