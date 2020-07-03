---
layout: post
title: Microsoft Ignite 2017 Recap - Subjective View
date: '2017-10-03T12:35:00.000-07:00'
author: Alex Terentiev
tags:
- SharePoint Online
- MS Graph
- SharePoint 2016
- SPFx
- O365
- MSIgnite
- SharePoint Framework
- Office 365
- SharePoint Framework Extensions
- SharePoint
modified_time: '2017-10-03T15:48:48.198-07:00'
featured_image_thumbnail:
featured_image: assets/images/posts/2017/teams-integration.png
blogger_id: tag:blogger.com,1999:blog-3066084330774405472.post-2848638393997482151
blogger_orig_url: http://blog.aterentiev.com/2017/10/microsoft-ignite-2017-recap-subjective.html
---

In this post I want to recap announcements that were done during Microsoft Ignite 2017.<br />It is not a full list but more an excerpt of features that are important to myself and our company. I will use logical structure on announcements provided by Microsoft. It's based on processes in the organization, not this or that product or technology.<br />I will also add some tags to announcements that may help to select the most valuable ones for yourself. <a name='more'></a><h2>1. Share and work together</h2><h3>1.1. New file thumbnails and previews</h3>Users can preview 270+ file types right in document libraries and also search results. Developers can also request thumbnails and previews using Microsoft Graph API.<br/><img border="0" src="{{site.baseurl}}/assets/images/posts/2017/2017-10-03-1.png" /><br /><div class="imgSource">Source - <a href="https://techcommunity.microsoft.com/t5/SharePoint-Blog/Connecting-the-modern-workplace-with-SharePoint-and-OneDrive/ba-p/110399" target="_blank">The SharePoint Community blog</a></div><b>When to expect:</b> Released.<br /><b>Tags:</b> SharePoint, SharePoint Document Library, OneDrive, Microsoft Graph, Microsoft Teams <h3>1.2. Connect existing sites to Office 365 Groups ("Groupify" a team site)</h3>Users can create an Office 365 Group for an existing site. A group is created with a shared inbox for conversations, a shared calendar, Planner to manage tasks, and the option to connect a team in Microsoft Teams.<br />More information can be found <a href="https://techcommunity.microsoft.com/t5/SharePoint-Blog/Work-better-together-with-SharePoint-team-sites-Office-365-app/ba-p/109550" target="_blank">here</a><br /><b>When to expect:</b> coming soon.<br /><b>Tags:</b> SharePoint, SharePoint Site, Office 365 Groups, Microsoft Teams <h3>1.3. Deeper integration with Microsoft Teams</h3>Users can preview 270+ file types right inside Microsoft Teams, and more easily add SharePoint pages, including your team site’s home page, as tabs in Microsoft Teams.<br/><img border="0" src="{{site.baseurl}}/assets/images/posts/2017/teams-integration.png" /><br /><div class="imgSource">Source - <a href="https://techcommunity.microsoft.com/t5/SharePoint-Blog/Connecting-the-modern-workplace-with-SharePoint-and-OneDrive/ba-p/110399" target="_blank">The SharePoint Community blog</a></div><b>When to expect:</b> coming soon.<br /><b>Tags:</b> SharePoint, Microsoft Teams <h3>1.4. The SharePoint Migration Tool</h3>Users or administrators can automate the migration of files from file servers and on-premises SharePoint document libraries to Office 365. <a href="https://techcommunity.microsoft.com/t5/SharePoint-Blog/Introducing-the-SharePoint-Migration-Tool-from-Microsoft/ba-p/109767" target="_blank">More information.</a><br /><img border="0" src="{{site.baseurl}}/assets/images/posts/2017/migration tool.png" /><div class="imgSource">Source - <a href="https://techcommunity.microsoft.com/t5/SharePoint-Blog/Introducing-the-SharePoint-Migration-Tool-from-Microsoft/ba-p/109767" target="_blank">The SharePoint Community blog</a></div><b>When to expect:</b> coming soon.<br /><b>Tags:</b> SharePoint, Microsoft Teams <h2>2. Inform and engage people</h2><h3>2.1. New 1st party web parts</h3>New and updated 1st party web parts to build rich, more dynamic pages:<br /><b>NEW</b><ul><li>Planner - View Planner plans inside SharePoint pages and news with visual Kanban task layouts and beautiful plan overviews.</li><li>Microsoft Forms – Create your survey at forms.office.com, grab the final Share URL and display your surveys right within the SharePoint user interface. You can choose to show the results after the user submits the form, too.</li><li>Group calendar – display your Office 365 group’s upcoming and past calendar meetings and events.</li><li>File viewer – Beautifully, visually highlight over 270+ file types from within SharePoint pages and news article. This web part is an update (+ name change) to the Document web part, and continues to support embedding Word, Excel and PPT, and now renders PDFs, 3D models, medical images, and more.</li><li>Twitter – bring in live tweets from chosen Twitter handles, specific collections or via search keywords. It’s always nice to show the live context of what’s being said externally right alongside the context of what you are working on internally.</li><li>Spacer & Divider – simple web parts that give you the ability to add physical space between web parts, or a visual line in between.</li></ul><b>UPDATED</b><ul><li>Yammer – Now out of preview, the Yammer web part can be programmed to showcase a Yammer group discussion, and it looks great in the Web layout of SharePoint pages and news articles and when viewed within the SharePoint mobile app (iOS, Android and Windows Mobile).</li><li>Image – You can pull in Bing Images that utilize the Creative Common license to enhance your pages and news (you are prompted to review the image licensing). You, too, can pull images directly from a specific document library. And once your images are on the page, you can edit them inline with simple gestures like adjusting the ratio and cropping.</li><li>Text – the rich text editor web part now gives you greater control for how your text appears. From the simple command bar within the web part, select the ellipses to display the broader set of choices in the edit pane, like font colors and highlights, plus table creation and editing.</li><li>People – a new Descriptive display shows more profile information with room to add custom links and descriptions per person.</li><li>Events – greater control of the preferred date range to show upcoming events, plus the ability to see each instance of recurring events.</li><li>Highlighted content – Ability to choose from a specific document library as the source, more design and layout choices like Filmstrip and Masonry, plus additional filtering mechanism to refine by document type and control metadata mapping to influence the display of the search-based results within the web part.</li></ul><a href="https://techcommunity.microsoft.com/t5/SharePoint-Blog/Refine-your-message-and-increase-your-reach-with-SharePoint/ba-p/109553">More about new and updated web parts</a><img border="0" src="{{site.baseurl}}/assets/images/posts/2017/webparts.png" /><div class="imgSource">Source - <a href="https://techcommunity.microsoft.com/t5/SharePoint-Blog/Refine-your-message-and-increase-your-reach-with-SharePoint/ba-p/109553" target="_blank">The SharePoint Community Blog</a></div><b>When to expect:</b> coming soon.<br /><b>Tags:</b> SharePoint, SharePoint Framework <h3>2.2. Customized themes, site scripts, site designs and page designs</h3>Ability manage the site theme and the layout of sites and pages. Ability to empower other users to create new sites based on custom site designs, provisioned with a theme, pages, and layouts suited to the business requirements of the site.<br /><a href="https://docs.microsoft.com/en-us/sharepoint/dev/declarative-customization/site-theming/sharepoint-site-theming-overview" target="_blank">More about custom themes</a><br/><img border="0" src="{{site.baseurl}}/assets/images/posts/2017/2017-10-03-5.png" /><div class="imgSource">Source - <a href="https://view.officeapps.live.com/op/embed.aspx?src=https%3A%2F%2F8gportalvhdsf9v440s15hrt.blob.core.windows.net%2Fignite2017%2Fsession-presentations%2FBRK3354.PPTX" target="_blank">BRK23354 - Using custom themes and designs to standardize the creation of clean, functional SharePoint sites by Sean Squires</a></div><b>When to expect:</b> Site themes are released. Site scripts and site designs are expected in late 2017.<br/><b>Tags:</b> SharePoint <h3>2.3. SharePoint Hub sites</h3>SharePoint hub sites will allow to bring together related sites, combine news and activities, to scope and simplify search and make related sites to have shared navigation and look-and-feel. Because sites can be re-arranged as organization needs change, SharePoint hub sites allow re-associate the sites when needed.<br /><a href="https://techcommunity.microsoft.com/t5/SharePoint-Blog/SharePoint-hub-sites-new-in-Office-365/ba-p/109547" target="_blank">More information on hub sites</a><br/><img border="0" src="{{site.baseurl}}/assets/images/posts/2017/hub%2Bsites.png" /><div class="imgSource">Source - <a href="https://techcommunity.microsoft.com/t5/SharePoint-Blog/SharePoint-hub-sites-new-in-Office-365/ba-p/109547" target="_blank">The SharePoint Community blog</a></div><b>When to expect:</b> Q2 2018.<br/><b>Tags:</b> SharePoint <h3>2.4. Updated web parts toolbox</h3>Web parts toolbox got expanded view, search capability, ability to sort by category or by alpha. It also includes data connectors and SharePoint Add-ins.<br /><img border="0" src="{{site.baseurl}}/assets/images/posts/2017/webparts1.png" /><div class="imgSource">Source - <a href="https://techcommunity.microsoft.com/t5/SharePoint-Blog/Refine-your-message-and-increase-your-reach-with-SharePoint/ba-p/109553" target="_blank">The SharePoint Community Blog</a></div><b>When to expect:</b> Search is released. The rest of functionality is coming soon.<br/><b>Tags:</b> SharePoint <h3>2.5. Start with a copy of this page</h3>SharePoint team is adding an ability to create new page as a copy of existing page - very helpful feature for creating pages with the same structure but different content. User can Click New on current page and select Start with a copy of this page.<br /><b>When to expect:</b> First release by end of 2017.<br/><b>Tags:</b> SharePoint <h2>3. Transform business processes</h2><h3>3.1. Declarative column formatting</h3>Ability to add declarative JSON-based formatting the list/library columns. Allows to change column's look-and-feel based on simple rules and formulas. No JavaScript logic allowed.<br /><img border="0" src="{{site.baseurl}}/assets/images/posts/2017/custom formats.png" /><div class="imgSource">Source - <a href="https://view.officeapps.live.com/op/embed.aspx?src=https%3A%2F%2F8gportalvhdsf9v440s15hrt.blob.core.windows.net%2Fignite2017%2Fsession-presentations%2FBRK3252.PPTX" target="_blank">BRK3252 - Geek out with the product team on SharePoint lists and libraries by Lincoln DeMaris</a></div><b>When to expect:</b> October 2017.<br/><b>Tags:</b> SharePoint <h3>3.2. Custom forms built with PowerApps</h3>It was foreseen that PowerApps usage will be expanded as users needed InfoPath replacement. PowerApps will allow to create modern, interactive and functional forms.<br /><img border="0" src="{{site.baseurl}}/assets/images/posts/2017/powerapps.png" /><div class="imgSource">Source - <a href="https://techcommunity.microsoft.com/t5/SharePoint-Blog/Connecting-the-modern-workplace-with-SharePoint-and-OneDrive/ba-p/110399" target="_blank">The SharePoint Community blog</a></div><b>When to expect:</b> coming soon.<br/><b>Tags:</b> SharePoint, PowerApps <h3>3.3. Large lists and libraries support</h3>SharePoint team have added predictive indexing, which automatically adds indices to lists and libraries. It allows to store up to 30M of items in the list. It also allows not to be throttled when sorting or filtering large lists.<br /><b>When to expect:</b> Background indexing for lists of size 2500-19999 items is released. Predictive indexing for lists of size 20K+ will be released next year.<br/><b>Tags:</b> SharePoint <h3>3.4. Attention view</h3>The view helps users to focus on files that are pending, incomplete, etc.<br /><img border="0" src="{{site.baseurl}}/assets/images/posts/2017/attention.png" /><div class="imgSource">Source - <a href="https://view.officeapps.live.com/op/embed.aspx?src=https%3A%2F%2F8gportalvhdsf9v440s15hrt.blob.core.windows.net%2Fignite2017%2Fsession-presentations%2FBRK3252.PPTX" target="_blank">BRK3252 - Geek out with the product team on SharePoint lists and libraries by Lincoln DeMaris</a></div><b>When to expect:</b> coming soon (probably).<br/><b>Tags:</b> SharePoint <h3>3.5. Bulk editing in views and libraries</h3>First option is Details Pane-based editing when a user select multiple files/items and able to change their properties from Details Pane<br />Second option is Quick Edit implementation in Modern UI<br /><b>When to expect:</b> In a backlog.<br/><b>Tags:</b> SharePoint <h3>3.6. Filter Pane</h3>Filters pane is designed to replace Metadata Navigation and Filtering from Classic UI and also provide more responsive UI for filtering.<br />Any column in the list/library can be pinned to the filters pane.<br /><img border="0" src="{{site.baseurl}}/assets/images/posts/2017/filters.png" /><br /><b>When to expect:</b> Released.<br /><b>Tags:</b> SharePoint <h2>4. Harness collective knowledge</h2><h3>4.1. Personalized search results and updated search results page</h3>New search engine is intelligent and personalized. It provides results based on users' activity and collaboration inside the organization.<br />New search results page is designed to provide results in more structured way. And it also allows to preview files inline - user can, for example, view all presentation slides right inside the row in search results.<br /><b>When to expect:</b> coming soon.<br /><b>Tags:</b> SharePoint <h3>4.2. Text recognition for images</h3>Office 365 will extract text from images and also index it to be available in search results.<br /><b>When to expect:</b> coming soon.<br /><b>Tags:</b> SharePoint, OneDrive, Office 365 <h2>5. Protect and manage content</h2><h3>5.1. New SharePoint Admin center</h3>New admin center provides more simple administration of SharePoint tenant. It also contains some new features that were not available in previous implementation (as example, management of modern team sites and communication sites).<br /><a href="https://techcommunity.microsoft.com/t5/SharePoint-Blog/Introducing-the-new-SharePoint-Admin-Center/ba-p/109761" target="_blank">More information on admin center</a><br /><img border="0" src="{{site.baseurl}}/assets/images/posts/2017/admin-center.png" /><div class="imgSource">Source - <a href="https://techcommunity.microsoft.com/t5/SharePoint-Blog/Introducing-the-new-SharePoint-Admin-Center/ba-p/109761" target="_blank">The SharePoint Community blog</a></div><b>When to expect:</b> later this year.<br /><b>Tags:</b> SharePoint, SharePoint administration <h2>6. Extend and develop integrations and apps</h2><h3>6.1 SharePoint Framework Extensions are in GA</h3>Application Customizer, Command Set Customizer and Field Customizer are in GA and available for custom development across the world.<br /><a href="https://docs.microsoft.com/en-us/sharepoint/dev/spfx/extensions/overview-extensions" target="_blank">More information about Extensions</a><br /><img border="0" src="{{site.baseurl}}/assets/images/posts/2017/extensions.png" /><br /><b>When to expect:</b> Released.<br /><b>Tags:</b> SharePoint, SharePoint Framework, SharePoint Development <h3>6.2. File Handlers v2 for OneDrive and SharePoint Document Libraries</h3>File Handlers engine allows developers to provide custom behaviors for any types of files (preview, additional actions when file or multiple files are selected) and even for custom file types as well.<br /><a href="https://docs.microsoft.com/en-us/onedrive/developer/file-handlers/">More info about File Handlers</a><br /><b>When to expect:</b> Released.<br /><b>Tags:</b> SharePoint, OneDrive, SharePoint Development, OneDrive Development, Microsoft Graph <h3>6.3. Microsoft Graph API for SharePoint lists</h3>Now API for SharePoint lists and content types in released and available for usage in all kinds of applications (SPFx solutions, SharePoint Add-ins, custom apps)<br /><a href="https://developer.microsoft.com/en-us/graph">More about Microsoft Graph</a><br /><a href="https://docs.microsoft.com/en-us/sharepoint/dev/spfx/overview-graphhttpclient" target="_blank">MS Graph support in SPFx</a><br /><b>When to expect:</b> Released.<br /><b>Tags:</b> SharePoint, SharePoint Framework, SharePoint Development, Microsoft Graph <h3>6.4. SharePoint 2016 Feature Pack 2 with SPFx web parts support</h3>The latest Feature Pack for SharePoint 2016 includes support of SharePoint Framework version 1.1. Note that SharePoint 2016 doesn't support Microsoft Graph<br />More information about  SPFx support in SharePoint 2016 can be found <a href="https://docs.microsoft.com/en-us/sharepoint/dev/spfx/sharepoint-2016-support" target="_blank">here</a><br /><b>When to expect:</b> Released.<br /><b>Tags:</b> SharePoint, SharePoint Framework, SharePoint Development <h3>6.5. Tenant-wide deployment of SharePoint Framework solutions</h3>SPFx solutions (unlike SharePoint Add-ins) now can be globally installed to all site collections in the tenant. Solution's developer should configure the solution to be "tenant-wide deployable" and tenant admin should approve it while deploying it to App Catalog. After that the solution will be available on all site collections in the tenant.<br />Restrictions: tenant-wide deployment is not applicable for solutions that are using Feature Framework to provision any assets to site collections.<br />More about tenant-wide deployment - <a href="https://docs.microsoft.com/en-us/sharepoint/dev/spfx/tenant-scoped-deployment" target="_blank">here</a><br/><b>When to expect:</b> Released.<br /><b>Tags:</b> SharePoint, SharePoint Framework, SharePoint Development <h3>6.6. SPFx tenant properties</h3>Tenant properties allows tenant administrators to add properties in the app catalog that can be read by various SharePoint Framework components. The tenant properties are managed by tenant administrators using the Microsoft SharePoint Online Management Shell which is a PowerShell module to manage your SharePoint Online subscription in the Office 365.<br/>More about tenant properties - <a href="https://docs.microsoft.com/en-us/sharepoint/dev/spfx/tenant-properties" target="_blank">here</a><b>When to expect:</b> available in First Release tenants.<br /><b>Tags:</b> SharePoint, SharePoint Framework, SharePoint Development <h3>6.7. Support of static Office UI Fabric styles</h3>Ability to use static styles from Office UI Fabric in custom web parts and property pane controls.<br/><b>When to expect:</b> coming soon.<br /><b>Tags:</b> SharePoint, SharePoint Framework, SharePoint Development <h3>6.8. Configuring permission scopes and access to 3d party Web API</h3>Tenant admins will be able to set custom permission scopes and also “trust” 3d party Web APIs. Developers specify Web APIs and scopes they need, tenant administrators are notified during the deployment and approve endpoints and scopes (similar to Add-ins and any other AAD apps)<br /><b>When to expect:</b> coming soon.<br /><b>Tags:</b> SharePoint, SharePoint Framework, SharePoint Development <h3>6.9. Apps permissions management user interface</h3>Tenant scoped UI to manage permissions for the apps<br/><b>When to expect:</b> - <br /><b>Tags:</b> SharePoint, SharePoint Framework, SharePoint Development <h3>6.10. ALM API for SPFx solutions</h3>API to allow targeted deployment, provisioning of SPFx solutions<br /><b>When to expect:</b> coming soon <br /><b>Tags:</b> SharePoint, SharePoint Framework, SharePoint Development <h3>6.11. Site collection App Catalog</h3>Tenant admin can allow to have local app catalog of SPFx solutions for a site. This feature is very important for development and testing of solutions as it allows to have dev and QA site collections instead of dev-QA tenants.<br /><b>When to expect:</b> coming soon <br /><b>Tags:</b> SharePoint, SharePoint Framework, SharePoint Development <h3>6.12. More Microsoft Graph support in future</h3>Microsoft Graph API for SharePoint will be expanded to contain more and more endpoints.<br />My understanding is that MS Graph should replace SharePoint REST API completely and also include (or partially include) functionality that is available in CSOM but not available in SharePoint REST API.<br /><a href="https://docs.microsoft.com/en-us/sharepoint/dev/spfx/overview-graphhttpclient" target="_blank">MS Graph support in SPFx</a><br /><b>When to expect:</b> iterative process of adding new endpoints<br /><b>Tags:</b> SharePoint, SharePoint Framework, SharePoint Development, Microsoft Graph <h3>6.13. SharePoint Patterns and Practices reusable controls</h3>Controls allow direct integration with SharePoint sites and data, and provide a library of extended control samples developers can use within their projects. The set of controls includes a set of pickers for Lists, People, and Terms, as well as a consistent site breadcrumb.<br />There are two projects for reusable controls, including one focused on the React framework along with a repository focused on core property JavaScript-based controls. These controls are currently in preview - making this a great opportunity to explore the controls, provide feedback, and maybe even make a pull-request.<br/><a href="https://dev.office.com/blogs/patterns-and-practices-updates-at-microsoft-ignite-2017" target="_blank">More about PnP controls</a><br /><a href="https://github.com/SharePoint/sp-dev-fx-property-controls" target="_blank">GitHub repo for Property Pane controls</a><br/><a href="https://github.com/SharePoint/sp-dev-fx-controls-react" target="_blank">GitHub repo for React controls</a><br/><b>When to expect:</b> developer preview<br /><b>Tags:</b> SharePoint, SharePoint Framework, SharePoint Development <h3>6.14. SharePoint Patterns and Practices Community Solutions</h3>PnP Community Solutions provide end-to-end user scenario samples that can provide inspiration for new custom solutions. Featuring an initial set of four web part projects that cover scenarios such as Contact Management, Inventory Tracking, and Change Requests, these web parts will make compelling foundations for your own solutions. These web parts also demonstrate various Patterns and Practices, including usage of multiple SharePoint lists, how you can extend SharePoint solutions with your own customizations, and integration with tools such as Microsoft Flow.<br /><a href="https://dev.office.com/blogs/patterns-and-practices-updates-at-microsoft-ignite-2017" target="_blank">More about PnP Community solutions</a><br /><a href="https://github.com/SharePoint/sp-dev-solutions">GitHub repo for solutions</a><br /><img border="0" src="{{site.baseurl}}/assets/images/posts/2017/inventorycheckout.png" /><div class="imgSource">Source - <a href="https://dev.office.com/blogs/patterns-and-practices-updates-at-microsoft-ignite-2017" target="_blank">Office Dev Center Blogs</a></div><b>When to expect:</b> developer preview<br /><b>Tags:</b> SharePoint, SharePoint Framework, SharePoint Development <h3>6.15. sp-pnp-js v3.0 release</h3>New release contains fluent API for SP REST API and Graph access. With the release of SharePoint Server 2016 Feature Pack 2, organizations can now create modern client-side web parts using SharePoint Framework. Updates to the sp-pnp-js now help customers more easily leverage SharePoint REST APIs in a consistent manner across both on-premises and Office 365 environments, helping ensure your solutions are seamlessly prepared for the cloud.<br /><a href="https://github.com/SharePoint/PnP-JS-Core" target="_blank">GitHub repo for sp-pnp-js</a><br /><b>When to expect:</b> released<br /><b>Tags:</b> SharePoint, SharePoint Framework, SharePoint Development <h3>6.16. SharePoint Provisioning Engine Improvements</h3>The SharePoint Provisioning engine has long been a standard tool for SharePoint developers who wish to provide tailored sites to users. With recent updates to the SharePoint Provisioning codebase, developers can now fully provision end-to-end experiences in modern sites, in addition to strong support for provisioning classic sites. <a href="https://github.com/sharepoint/pnp-sites-core" target="_blank">GitHub repo for PnP Sites provisioning core</a><br /><h3>6.17. New scoped set of npm packages for Patterns and Practices</h3>PnP now has a set of scoped npm packages - all of them available under @pnp scope.<br />Developers can simply select which package to use in their solutions, no need to install the whole PnP library.<br /><a href="https://www.npmjs.com/~officedevpnp">List of PnP npm packages</a><br /><b>When to expect:</b> released<br /><b>Tags:</b> SharePoint, SharePoint Framework, SharePoint Development <br /><br />One more announce - there will be a <a href="https://sharepointna.com/#!/" target="_blank">SharePoint Conference in Las Vegas</a> in May 2018! That's cool!<br /><img border="0" src="{{site.baseurl}}/assets/images/posts/2017/SPC-2018.png" /><br />I believe that's it. Please, feel free to add comments if you think there are more important announcements or you have more info on release dates, etc.<br />Have fun!