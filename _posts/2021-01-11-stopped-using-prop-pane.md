---
layout: post
title: Why I stopped using SharePoint Framework Property Pane
description: Review why I stopped using SharePoint Framework Property Pane and implement a custom property panel instead
tags:
  - M365
  - MS Teams
  - Microsoft
  - Microsoft 365
  - Microsoft Teams
  - O365
  - Office 365
  - SharePoint Online
  - Teams
  - SharePoint Framework
  - SPFx
  - SharePoint Framework Web Part
  - SPFx Web Part
  - SharePoint Framework Property Pane
  - SPFx Property Pane
  - Property Pane
featured_image: assets/images/posts/2021/2021-01-11-prop-pane.png
featured_image_thumbnail: assets/images/posts/2021/2021-01-11-prop-pane.png
slug: stopped-sharepoint-framework-property-pane
date: 2021-01-11T22:46:50.539Z
---
SharePoint Framework Web Parts have a great way to implement a user interface to configure web part properties - Web Part Property Pane. With a set of property pane controls available out-of-the-box, this functionality fits most of the developers' needs. But there are some limitations and subjective concerns why I decided to stop using OOB Property Pane and implement custom property panels for my web parts.

## Complex-ish Implementation of Custom Property Pane Controls
There are a lot of [property pane controls](https://docs.microsoft.com/en-us/sharepoint/dev/spfx/web-parts/basics/integrate-with-property-pane#property-pane-fields) available out-of-the-box. But sooner or later a developer finds himself in a situation when he needs to implement custom property pane control. 
SPFx provides [ability](https://docs.microsoft.com/en-us/sharepoint/dev/spfx/web-parts/guidance/build-custom-property-pane-controls) to create custom property pane controls and use them in web parts. But the implementation is more complex than a standard control as you need to implement the control itself (UI), and a contract, or mediator, that will be used in the web part to instantiate the control.
And you also need to correctly update a web part's properties from your control implementation as well.

## More Limited Set of Reusable Controls
Microsoft 365 PnP initiative has a great repo with custom [Reusable Property Pane Controls](https://pnp.github.io/sp-dev-fx-property-controls/). But, probably, this is the only source of reusable controls for property pane.
If you want to use some other available controls (for example, Fluent UI controls) you will need to implement a wrapper to make them work in the property pane.
But if you go with "standard" controls and custom property panel - you can use any available control/component you want. And you'll just need to implement a way to get the property/value from the controls and set it in the web part.

## Web Part Property Pane is not Available in Microsoft Teams Personal Apps
This is the most important reason why property pane may not fit your needs.
If you decide to implement Microsoft Teams Personal App using SPFx you should be aware that **web part's property pane will not be available for you.** (This statement is also true for the web part's properties, so you'll need to implement them by yourself. [Here](https://blog.aterentiev.com/teams-personal-app-configuration) you can find an example how to achieve it using user's OneDrive). Basically, it means that you will need to implement custom property panel anyway if you need your web part to work as Personal App. And why would you have 2 implementations if you can just use custom one everywhere? :)

## Conclusion
Web Part Property Pane is a great and easy way to make you solutions configurable and implement basic UI for the configuration.
But if you need custom components, want to use available community components for the configuration, or implement the web part to be available as an MS Teams Personal App - consider implementing custom property panel instead.


That's all for today!<br />
Have fun!
