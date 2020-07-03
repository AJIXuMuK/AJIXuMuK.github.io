---
layout: post
title: 'Step by Step: Configure Bot to Work in Teams and with Microsoft Graph'
date: '2019-07-10T14:00:00.001-07:00'
author: Alex Terentiev
tags:
- MS Graph
- C#
- Office 365
- Bot
- Instructions
- Microsoft Graph
- Azure
- Office Development
- ASP.NET Core
- Microsoft Teams
- BotFramework
- MS Teams
- O365
- Authentication
- Azure AD
modified_time: '2019-07-15T11:19:41.134-07:00'
featured_image_thumbnail: assets/images/posts/2019/teams-bot.png
featured_image: assets/images/posts/2019/teams-bot.png
blogger_id: tag:blogger.com,1999:blog-3066084330774405472.post-5921984543635999504
blogger_orig_url: http://blog.aterentiev.com/2019/07/step-by-step-configure-bot-to-work-in.html
---

This blog post is a step-by-step instruction on how to create a Bot from scratch using Microsoft Bot Framework v4, configure it to work with Microsoft Teams, and authenticate it to make Microsoft Graph requests.<br />There are a lot of resources around bots, authentication in bots and Microsoft Teams (and I'll list resources at the end of the post). But during my investigation I couldn't find an example or an article that walks through all 3 topics at the same time. And especially "from scratch" which is important if you want to understand the technology and the flow.<br />So this post is kind of a cheat sheet for myself, and hopefully for others.<br /><b>All the steps described are for Bot Framework v4 and ASP.NET Core.</b><br />Code is available <a href="https://github.com/AJIXuMuK/BotFramework/tree/master/TeamsGraphBot" target="_blank">here</a>.<br />  Keep in mind that except of code there are different configurations to be done in Azure and MS Teams.<br />So, it might worth reading the post. <br /><a name='more'></a><br /><h2>Table of Contents</h2><ol><li><a href="#prerequisites">Prerequisites</a></li><li><a href="#what-to-install">What to Install</a></li><li><a href="#create-empty-bot">Create Empty Bot</a></li><li><a href="#prep-azure-resources">Prepare Azure Resources</a></li><li><a href="#app-id-bot-config">Set AppId and App Password in Bot's Configuration</a></li><li><a href="#deployment">Deployment</a></li><li><a href="#test-bot-emulator">Test the Bot in Bot Framework Emulator</a></li><li><a href="#connect-teams">Connecting the Bot with Microsoft Teams</a></li><li><a href="#state-dialogs">State and Dialogs</a></li><li><a href="#auth">Authentication Time!</a></li><li><a href="#ms-graph">Add MS Graph Logic</a></li><li><a href="#next">Next Steps</a></li><li><a href="#references">References</a></li><li><a href="#conclusion">Conclusion</a></li></ol><a name="prerequisites"></a><h2>Prerequisites</h2>The bot to be created will be registered and hosted in Azure, work with Microsoft Graph from Microsoft Teams, and implemented using ASP.NET Core...<br />Saying all that the prerequisites are: <ol><li>O365 Tenant</li><li>Azure Subscription with Azure Bot Service, App Service</li><li>Visual Studio</li></ol><br /><a name="what-to-install"></a><h2>What to Install</h2><ol><li><a href="https://marketplace.visualstudio.com/items?itemName=BotBuilder.botbuilderv4" target="_blank">Bot Framework v4 SDK Templates for Visual Studio</a></li><li><a href="https://docs.microsoft.com/en-us/cli/azure/?view=azure-cli-latest" target="_blank">Azure CLI</a></li><li><a href="https://github.com/Microsoft/BotFramework-Emulator/blob/master/README.md" target="_blank">Bot Framework Emulator</a></li><li><a href="https://ngrok.com/" target="_blank">ngrok</a></li></ol><br /><a name="create-empty-bot"></a><h2>Create Empty Bot</h2><ol><li>Open Visual Studio to create a new project</li><li>Select "Empty Bot (Bot Framework v4)" project type<br /><img border="0" src="{{site.baseurl}}/assets/images/posts/2019/empty-bot.png" /></li><li>Type a name for the project, and select a folder. I'll be using <span class="code">TeamsGraphBot</span> name</li></ol>After these easy steps you already have a working bot that welcomes new users in the conversation with "Hello world!" phrase.<br />If you start the project (F5 in Visual Studio). You'll see a web page illustrating how to test the bot.<br /><img border="0" src="{{site.baseurl}}/assets/images/posts/2019/bot-is-ready.png" /><br /><br /><a name="prep-azure-resources"></a><h2>Prepare Azure Resources</h2><a href="https://docs.microsoft.com/en-us/azure/bot-service/bot-builder-deploy-az-cli?view=azure-bot-service-4.0&tabs=newrg#login-to-azure" target="_blank">Official documentation</a><br /><h3>Login and Connect to Azure Subscription</h3><ol><li>Open command prompt</li><li>Enter the command below to log in to Azure Portal: 
<div markdown="1">
{% highlight console %}

az login

{% endhighlight %}
</div>
It will open a browser window, allowing you to log in. </li><li>Next, set the subscription to use: 
<div markdown="1">
{% highlight console %}

az account set --subscription "<azure-subscription-id>"

{% endhighlight %}
</div>
If you are not sure which subscription to use for deploying the bot, you can view the list of subscriptions for your account by using <span class="code">az account list</span> command. </li></ol><h3>Register Azure AD Application</h3>Next step is to register Azure AD App. It can be done either using Azure CLI as described in the documentation listed above, or using Azure Portal.<br />For Azure CLI registration use the command below: 
<div markdown="1">
{% highlight console %}

az ad app create --display-name "displayName" --password "AtLeastSixteenCharacters_0" --available-to-other-tenants

{% endhighlight %}
</div>
where  <ul><li><span class="code">displayName</span> is a name for the application, </li><li><span class="code">password</span> is a 'client secret'.The password must be at least 16 characters long, contain at least 1 upper or lower case alphabetical character, and contain at least 1 special character </li><li><span class="code">available-to-other-tenants</span> defines that the application can be used from any Azure AD tenant. This must be <span class="code">true</span> to enable your bot to work with the Azure Bot Service channels. </li></ul><br />If you go with Azure Portal UI, do the next things while registering the App: <ol><li>When registering the app, select <b>Account in any organizational directory and personal Microsoft accounts (e.g. Skype, Xbox, Outlook.com)</b>:<br /><img border="0" src="{{site.baseurl}}/assets/images/posts/2019/register-ad-app.png" /></li><li>Navigate to <b>Certificates &amp; secrets</b>, create new client secret that never expires, and store it somewhere:<br /><img border="0" src="{{site.baseurl}}/assets/images/posts/2019/add-secret.png" /></li></ol><br /><h3>Create Azure Resources</h3>The bot can be deployed in a new or existing Azure resource group. I will use new group here.<br />To create all bot resources in new resource group: <ol><li>In command prompt navigate to <span class="code">DeploymentTemplates</span> folder inside the bot project</li><li>Run the command below: 
<div markdown="1">
{% highlight console %}

az deployment create --name "<name-of-deployment>" --template-file "template-with-new-rg.json" --location "location-name" --parameters appId="<appId>" appSecret="<appSecret>" botId="<id-or-name-of-bot>" botSku=F0 newAppServicePlanName="<name-of-app-service-plan>" newWebAppName="<name-of-web-app>" groupName="<new-group-name>" groupLocation="<location>" newAppServicePlanLocation="<location>"

{% endhighlight %}
</div>
where <ul><li><span class="code">name</span> - Friendly name for the deployment. </li><li><span class="code">template-file</span> - The path to the template. We use <span class="code">template-with-new-rg.json</span> file provided in the <span class="code">DeploymentTemplates</span> folder of the project. </li><li><span class="code">location</span> - Location. Values from: <span class="code">az account list-locations</span>. For example, "West US", or "Central US". </li><li><span class="code">parameters</span> - Provide deployment parameter values. <span class="code">appId</span> and <span class="code">appSecret</span> values are AppId and Client Secret of the Azure AD App from previous step. The <span class="code">botId</span> parameter should be globally unique and is used as the immutable bot ID. It is also used to configure the display name of the bot, which is mutable. <span class="code">botSku</span> is the pricing tier and can be F0 (Free) or S1 (Standard). <span class="code">newAppServicePlanName</span> is the name of App Service Plan. <span class="code">newWebAppName</span> is the name of the Web App you are creating. <span class=""code">groupName</span> is the name of the Azure resource group you are creating. <span class="code">groupLocation</span> is the location of the Azure resource group. <span class="code">newAppServicePlanLocation</span> is the location of the App Service Plan. </li></ul></li></ol><br /><a name="app-id-bot-config"></a><h2>Set AppId and App Password in Bot's Configuration</h2>After we prepared all the resources, and to be specific - registered new Azure AD App, we need to set new app's client Id and client Secret in the bot's project settings. These parameters are used to authenticate the bot to communicate with Bot Framework service. <ol><li>In Visual Studio open <span class="code">appsettings.json</span> file:<br /><img border="0" src="{{site.baseurl}}/assets/images/posts/2019/appsettings.png" /></li><li>Enter Azure AD App's client Id as <span class="code">MicrosoftAppId</span> value, and client secret as <span class="code">MicrosoftAppPassword</span> value:<br /><img border="0" src="{{site.baseurl}}/assets/images/posts/2019/appsettings-content.png" /></li></ol><br /><a name="deployment"></a><h2>Deployment</h2>The deployment can be done using Azure CLI and steps listed in the <a href="https://docs.microsoft.com/en-us/azure/bot-service/bot-builder-deploy-az-cli?view=azure-bot-service-4.0&tabs=newrg#login-to-azure" target="_blank">Official documentation</a> mentioned above.<br />But let's in this sample use old good Azure Publishing Profile.<br /><ol><li>Navigate to Azure Portal, to the newly created Bot Web App. You can go there either from App Service section, or go to the bot's resource group and start from there:<br /><img border="0" src="{{site.baseurl}}/assets/images/posts/2019/bot-resource-group.png" /></li><li>Click on <b>Get publish profile</b> to download Azure Publishing Profile to your machine:<br /><img border="0" src="{{site.baseurl}}/assets/images/posts/2019/bot-app-service.png" /></li><li>In Visual Studio right click on the bot's project -&gt; <b>Publish</b>:<br /><img border="0" src="{{site.baseurl}}/assets/images/posts/2019/publish.png" /></li><li>In the popup click <b>Import Profile...</b> in the bottom left corner:<br /><img border="0" src="{{site.baseurl}}/assets/images/posts/2019/import-profile.png" /></li><li>Change settings if you wish to. For debugging/development purposes we can change <b>Configuration</b> from Release to Debug. </li><li>Save the changes and click <b>Publish</b>:<br /><img border="0" src="{{site.baseurl}}/assets/images/posts/2019/publish-azure.png" /></li></ol>If everything goes right you should see a web page hosted on Azure that looks identical to the one we saw after hitting F5:<br /><img border="0" src="{{site.baseurl}}/assets/images/posts/2019/bot-is-ready-azure.png" /><br />The difference is that now our bot is hosted on Azure.<br /><br /><a name="test-bot-emulator"></a><h2>Test the Bot in Bot Framework Emulator</h2>Now we can debug our bot from Bot Framework Emulator. <ol><li>Start the project (F5 in Visual Studio). You'll see a web page illustrating how to debug the bot.<br /><img border="0" src="{{site.baseurl}}/assets/images/posts/2019/bot-is-ready.png" /></li><li>Copy <span class="code">localhost</span> URL. </li><li>Launch Bot Framework Emulator and click on <b>Open Bot</b> button<br /><img border="0" src="{{site.baseurl}}/assets/images/posts/2019/bot-framework-emulator.png" /></li><li>Enter copied <span class="code">localhost</span> in Bot URL input and hit <b>Connect</b><br /><img border="0" src="{{site.baseurl}}/assets/images/posts/2019/bot-url.png" /></li></ol>After that you'll see communication between emulator and the bot in LOG and as a result - "Hello world!" message from your bot:<br /><img border="0" src="{{site.baseurl}}/assets/images/posts/2019/test-bot.png" /><br />You can test Azure-hosted bot in Bot Framework Emulator as well in the same way: use created App Service web URL instead of <span class="code">localhost</span>. The only additional configuration to be done to allow remote testing in configuring the emulator to use ngrok tunelling. <ol><li>Click on cogwheel in left bottom corner of the emulator: <img border="0" src="{{site.baseurl}}/assets/images/posts/2019/emulator-settings.png" /></li><li>Provide path to ngrok and click <b>Save</b>. It will allow the emulator to automatically launch ngrok:<br /><a href="https://3.bp.blogspot.com/-RSECCbihfhU/XReLPuVK4wI/AAAAAAAABQ0/-jn4fo6qgIILbCPKSnUvaHh3PJr0afu2ACLcBGAs/s1600/Screen%2BShot%2B2019-06-29%2Bat%2B9.00.42%2BAM.png" imageanchor="1" ><img border="0" src="https://3.bp.blogspot.com/-RSECCbihfhU/XReLPuVK4wI/AAAAAAAABQ0/-jn4fo6qgIILbCPKSnUvaHh3PJr0afu2ACLcBGAs/s1600/Screen%2BShot%2B2019-06-29%2Bat%2B9.00.42%2BAM.png" data-original-width="1032" data-original-height="394" width="600" /></a></li></ol><br /><a name="connect-teams"></a><h2>Connecting the Bot with Microsoft Teams</h2>Next step is to connect the bot with MS Teams.<br />It can be easily done using <a href="https://github.com/OfficeDev/BotBuilder-MicrosoftTeams-dotnet" target="_blank">Bot Builder SDK 4 - Microsoft Teams Extensions</a>.<br /><br /><h3>Custom Bot Framework Adapter</h3>Currently we're using default <span class="code">BotFrameworkHttpAdapter</span> available in Bot Framework SDK. You can see that in <span class="code">Startup.cs</span>: 
<div markdown="1">
{% highlight csharp %}
// Create the Bot Framework Adapter.
services.AddSingleton<IBotFrameworkHttpAdapter, BotFrameworkHttpAdapter>();
{% endhighlight %}
</div>
Let's create custom Bot Framework Adapter. It allows us to take control over such parts of the flow as Middleware (see below) and error handling. <ol><li>Add <span class="code">AdapterWithErrorHandler</span> to the project</li><li>Copy the content of the class from EchoBot template: 
<div markdown="1">
{% highlight csharp %}

public class AdapterWithErrorHandler : BotFrameworkHttpAdapter
{
  public AdapterWithErrorHandler(ICredentialProvider credentialProvider, ILogger<BotFrameworkHttpAdapter> logger, ConversationState conversationState = null)
    : base(credentialProvider)
  {
    OnTurnError = async (turnContext, exception) =>
    {
      // Log any leaked exception from the application.
      logger.LogError($"Exception caught : {exception.Message}");

      // Send a catch-all apology to the user.
      await turnContext.SendActivityAsync("Sorry, it looks like something went wrong.");

      if (conversationState != null)
      {
        try
        {
          // Delete the conversationState for the current conversation to prevent the
          // bot from getting stuck in a error-loop caused by being in a bad state.
          // ConversationState should be thought of as similar to "cookie-state" in a Web pages.
          await conversationState.DeleteAsync(turnContext);
        }
        catch (Exception e)
        {
          logger.LogError($"Exception caught on attempting to Delete ConversationState : {e.Message}");
        }
      }
    };
  }
}

{% endhighlight %}
</div>
</li><li>Let's reference the Adapter in <span class="code">Startup.cs</span>:<br />Replace 
<div markdown="1">
{% highlight csharp %}

// Create the Bot Framework Adapter.
services.AddSingleton<IBotFrameworkHttpAdapter, BotFrameworkHttpAdapter>();

{% endhighlight %}
</div>
With 
<div markdown="1">
{% highlight csharp %}

// Create the Bot Framework Adapter.
services.AddSingleton<IBotFrameworkHttpAdapter, AdapterWithErrorHandler>();

{% endhighlight %}
</div>
</li></ol><br /><h3>Middleware</h3>Bot to MS Teams connection is based on <a href="https://docs.microsoft.com/en-us/azure/bot-service/bot-builder-concept-middleware?view=azure-bot-service-4.0" target="_blank">Middleware</a> concept. Middleware is simply a class that sits between the adapter and your bot logic, added to your adapter's middleware collection during initialization. The SDK allows you to write your own middleware or add middleware created by others. Every activity coming into or out of your bot flows through your middleware.<br />The adapter processes and directs incoming activities in through the bot middleware pipeline to your bot’s logic and then back out again. As each activity flows in and out of the bot, each piece of middleware can inspect or act upon the activity, both before and after the bot logic runs.<br />In case of MS Teams there is a <span class="code">TeamsMiddleware</span> implementation that processes activities to add Teams context in <span class="code">TurnState</span>.<br />So, let's add <span class="code">TeamsMiddleware</span> to our bot implementation. <ol><li>Add a reference to <a href="https://www.nuget.org/packages/Microsoft.Bot.Builder.Teams" target="_blank">Microsoft.Bot.Builder.Teams</a> NuGet package. Note: we need version 4.*. And currently the version is in prerelease. That's why you need to check "Include prerelease" checkbox in NuGet Package Manager while searching for the module:<br /><img border="0" src="{{site.baseurl}}/assets/images/posts/2019/nuget-teams.png" /></li><li>Add <span class="code">TeamsMiddleware</span> usage in <span class="code">AdapterWithErrorHandler</span> constructor: 
<div markdown="1">
{% highlight csharp %}

Use(new TeamsMiddleware(credentialProvider));

{% endhighlight %}
</div>
Full code of the constructor: 
<div markdown="1">
{% highlight csharp %}

public class AdapterWithErrorHandler : BotFrameworkHttpAdapter
{
  public AdapterWithErrorHandler(ICredentialProvider credentialProvider, ILogger<BotFrameworkHttpAdapter> logger, ConversationState conversationState = null)
    : base(credentialProvider)
  {
    OnTurnError = async (turnContext, exception) =>
    {
      // Log any leaked exception from the application.
      logger.LogError($"Exception caught : {exception.Message}");

      // Send a catch-all apology to the user.
      await turnContext.SendActivityAsync("Sorry, it looks like something went wrong.");

      if (conversationState != null)
      {
        try
        {
          // Delete the conversationState for the current conversation to prevent the
          // bot from getting stuck in a error-loop caused by being in a bad state.
          // ConversationState should be thought of as similar to "cookie-state" in a Web pages.
          await conversationState.DeleteAsync(turnContext);
        }
        catch (Exception e)
        {
          logger.LogError($"Exception caught on attempting to Delete ConversationState : {e.Message}");
        }
      }
    };

    Use(new TeamsMiddleware(credentialProvider));
  }
}

{% endhighlight %}
</div>
</li></ol><br /><h3>ITeamsContext</h3>Now we can get Teams context in the bot's turns (events).<br />Let's add some simple code to verify that the context is presented and we can get information from it. <ol><li>Go to the bot's code (in this sample - <span class="code">TeamsGraphBot.cs</span>) </li><li>Override <span class="code">OnMessageActivityAsync</span> method to react on user's messages </li><li>Add code to check for MS Teams context and to display some of the properties: 
<div markdown="1">
{% highlight csharp %}

protected override async Task OnMessageActivityAsync(ITurnContext<IMessageActivity> turnContext, CancellationToken cancellationToken)
{
  var teamsContext = turnContext.TurnState.Get<ITeamsContext>();

  if (teamsContext != null) // the bot is used inside MS Teams
  {
    if (teamsContext.Team != null) // inside team
    {
      await turnContext.SendActivityAsync(MessageFactory.Text($"Team Id: {teamsContext.Team.Id}"), cancellationToken).ConfigureAwait(false);
    }
    else // private or group chat
    {
      await turnContext.SendActivityAsync(MessageFactory.Text($"We're in MS Teams but not in Team"), cancellationToken).ConfigureAwait(false);
    }
  }
  else // outside MS Teams
  {
    await turnContext.SendActivityAsync(MessageFactory.Text("We're not in MS Teams context"), cancellationToken).ConfigureAwait(false);
  }
}

{% endhighlight %}
</div>
</li></ol>If we call our bot from Bot Emulator, or Web Chat we should see "We're not in MS Teams context" response from the bot.<br /><br /><h3>Publish</h3>Let's publish all the changes to Azure so we could use them later on from MS Teams. <br /><br /><h3>Add Microsoft Teams Channel to the Bot</h3>A channel is a connection between the bot and communication apps. You configure a bot to connect to the channels you want it to be available on. The Bot Framework Service, configured through the Azure portal, connects your bot to these channels and facilitates communication between your bot and the user. Web Chat channel is pre-configured for you.<br />Now we need to add Microsoft Teams channel to the bot. <ol><li>Sign in to <a href="https://portal.azure.com/" target="_blank">Azure Portal</a>.</li><li>Select the bot that you want to configure.</li><li>In the Bot Service blade, click <b>Channels</b> under <b>Bot Management</b>.</li><li>Click on Microsoft Teams icon to add MS Teams channel to the bot:<br /><img border="0" src="{{site.baseurl}}/assets/images/posts/2019/channels.png" /></li></ol><br /><h3>Add Teams App with the Bot</h3>Our bot is ready to work with MS Teams. But MS Teams knows nothing about the bot.<br />We need to register new app and provide information about our bot. It will allow Microsoft Teams to communicate with the bot. <ol><li>Go to MS Teams (either desktop or web client) </li><li>Open <b>App Studio</b>. If you're not familiar with App Studio you can read more about it <a href="https://docs.microsoft.com/en-us/microsoftteams/platform/get-started/get-started-app-studio" target="_blank">here</a>. </li><li>Navigate to <b>Manifest Editor</b> tab and click <b>Create a new app</b></li><li>Enter App Details:<br /><img border="0" src="{{site.baseurl}}/assets/images/posts/2019/teams-app-builder.png" /><br />The most important setting in App Details is <b>Short Name</b>. This name will be used to <b>@mention</b> bot in conversations. </li><li>Add bot information in <b>Capabilities</b> section:<br /><img border="0" src="{{site.baseurl}}/assets/images/posts/2019/teams-set-up-bot.png" /><br />where: <ul><li>We need to select <b>Existing bot</b> as we're going to connect to the bot that has been already create.</li><li><span class="code">Name</span> - Name of the bot.</li><li><span class="code">Bot Id</span> - App ID of the bot's Azure AD Application (created above). Here you can either enter the value manually, or select <b>Select from one of my existing bots</b> if the bot is registered in the Azure AD connected to the current tenant.</li><li>Select all the scopes you want. I selected all 3 to make the bot available in every conversation type.</li></ul>After hitting <b>Save</b> you'll see bot configuration page where you can generate new client secret (will be automatically propagated to Azure AD App), change messaging endpoint if the bot has some non-default configuration, and add commands that will be displayed to a user as a hint when messaging to the bot.<br />For this sample we can leave everything as is. </li><li>Add <span class="code">token.botframework.com</span> in the list of valid domains in <b>Domains and permissions section:</b><br /><img border="0" src="{{site.baseurl}}/assets/images/posts/2019/valid-domains.png" /><br />It will allow to implement authentication flow later on. </li><li>Go to <b>Test and distribute</b> section and click <b>Install</b></li><li>In the popup select <b>Add for you</b> as well as some team in <b>Add to a team or chat</b> and click <b>Install</b>. It will create a private chat with bot and will add the bot to the selected team:<br /><img border="0" src="{{site.baseurl}}/assets/images/posts/2019/teams-install-bot.png" /><br />Now the bot can be tested from the Team:<br /><img border="0" src="{{site.baseurl}}/assets/images/posts/2019/teams-bot.png" /><br />And from one-on-one chat:<br /><img border="0" src="{{site.baseurl}}/assets/images/posts/2019/personal-app-bot.png" /></li></ol><br /><a name="state-dialogs"></a><h2>State and Dialogs</h2>Next step in our journey is to add state and dialogs to the bot.<br /><br /><h3>State</h3>Our bot is just an asp.net core web application. And it is stateless by default, meaning that the bot doesn't know what happened in a previous round of the conversation and can't use this information, or data, for the next action.<br />But thankfully, we can add state support to our project.<br />There are 3 different types of state: <ul><li>User state - available in any turn that the bot is conversing with that user on that channel, regardless of the conversation.</li><li>Conversation state - available in any turn in a specific conversation, regardless of user (i.e. group conversations)</li><li>Private conversation state - scoped to both the specific conversation and to that specific user</li></ul>In our case we'll add User state to manage authentication, and Conversation state that is used by Dialogs engine.<br /><ol><li>As state should be stored somewhere, we need to add storage layer to the bot. For development purposes, we'll be using in-memory storage. But for production Microsoft recommends to use CosmosDB storage implementation. You can also create your own implementation of storage layer if needed.<br />So, let's add in-memory storage layer in <span class="code">Startup.cs ConfigureServices</span>: 
<div markdown="1">
{% highlight csharp %}

// storage
services.AddSingleton<IStorage, MemoryStorage>();

{% endhighlight %}
</div>
</li><li>Now we can add states singletons to our bot. It is also done in <span class="code">Startup.cs ConfigureServices</span>: 
<div markdown="1">
{% highlight csharp %}

// Create the User state. (Used in this bot's Dialog implementation.)
services.AddSingleton<UserState>();

// Create the Conversation state. (Used by the Dialog system itself.)
services.AddSingleton<ConversationState>();

{% endhighlight %}
</div>
</li></ol>Currently our <span class="code">ConfigureServices</span> method should look like this: 
<div markdown="1">
{% highlight csharp %}

// This method gets called by the runtime. Use this method to add services to the container.
public void ConfigureServices(IServiceCollection services)
{
  services.AddMvc().SetCompatibilityVersion(CompatibilityVersion.Version_2_1);

  // Create the credential provider to be used with the Bot Framework Adapter.
  services.AddSingleton<ICredentialProvider, ConfigurationCredentialProvider>();

  // Create the Bot Framework Adapter.
  services.AddSingleton<IBotFrameworkHttpAdapter, AdapterWithErrorHandler>();

  // storage
  services.AddSingleton<IStorage, MemoryStorage>();

  // Create the User state. (Used in this bot's Dialog implementation.)
  services.AddSingleton<UserState>();

  // Create the Conversation state. (Used by the Dialog system itself.)
  services.AddSingleton<ConversationState>();

  // Create the bot as a transient. In this case the ASP Controller is expecting an IBot.
  services.AddTransient<IBot, EmptyBot>();
}

{% endhighlight %}
</div>
<br /><h3>Dialogs</h3>Dialogs are structures in your bot that act like functions in your bot's program; each dialog is designed to perform a specific task, in a specific order.<br />You can also think of dialogs like "topics" in the conversation. For example, "how can I help" is the initial topic. User replies to bot's question, and the bot decides what other topic (dialog) to start (create support request, show current weather, etc.).<br />Please, read <a href="https://docs.microsoft.com/en-us/azure/bot-service/bot-builder-concept-dialog?view=azure-bot-service-4.0" target="_blank">official documentation</a> to be familiar with dialog concepts, such as <b>dialog sets</b>, <b>dialog context</b>, <b>dialog result</b>, and different types of dialogs available in the SDK.<br />Let's add dialogs support to our project and modify the bot to work with dialogs. <ol><li>First, let's install <a href="https://www.nuget.org/packages/Microsoft.Bot.Builder.Dialogs/" target="_blank">Microsoft.Bot.Builder.Dialogs</a> NuGet package</li><li>Next, create <span class="code">DialogExtensions.cs</span> class. This class appears in many samples and contains <span class="code">Run</span> extension method to correctly start new or continue existing dialog: 
<div markdown="1">
{% highlight csharp %}

public static class DialogExtensions
{
  public static async Task Run(this Dialog dialog, ITurnContext turnContext, IStatePropertyAccessor<DialogState> accessor, CancellationToken cancellationToken = default(CancellationToken))
  {
    var dialogSet = new DialogSet(accessor);
    dialogSet.Add(dialog);

    var dialogContext = await dialogSet.CreateContextAsync(turnContext, cancellationToken);

    var results = await dialogContext.ContinueDialogAsync(cancellationToken);
    if (results.Status == DialogTurnStatus.Empty)
    {
      await dialogContext.BeginDialogAsync(dialog.Id, null, cancellationToken);
    }
  }
}

{% endhighlight %}
</div>
</li><li>Next, let's create a generic base bot to work with dialogs. Let's call it <span class="code">DialogBot</span> and add it to the new <span class="code">Bots</span> folder: 
<div markdown="1">
{% highlight csharp %}

public class DialogBot<T> : ActivityHandler where T : Dialog
{
  protected readonly BotState ConversationState;
  protected readonly Dialog Dialog;
  protected readonly ILogger Logger;
  protected readonly BotState UserState;

  public DialogBot(ConversationState conversationState, UserState userState, T dialog, ILogger<DialogBot<T>> logger)
  {
    ConversationState = conversationState;
    UserState = userState;
    Dialog = dialog;
    Logger = logger;
  }

  public override async Task OnTurnAsync(ITurnContext turnContext, CancellationToken cancellationToken = default(CancellationToken))
  {
    if (turnContext?.Activity?.Type == ActivityTypes.Invoke && turnContext.Activity.ChannelId == "msteams")
      await Dialog.Run(turnContext, ConversationState.CreateProperty<DialogState>(nameof(DialogState)), cancellationToken);
    else
      await base.OnTurnAsync(turnContext, cancellationToken);

    // Save any state changes that might have occurred during the turn.
    await ConversationState.SaveChangesAsync(turnContext, false, cancellationToken);
    await UserState.SaveChangesAsync(turnContext, false, cancellationToken);
  }

  protected override async Task OnMessageActivityAsync(ITurnContext<IMessageActivity> turnContext, CancellationToken cancellationToken)
  {
    Logger.LogInformation("Running dialog with Message Activity.");

    // Run the Dialog with the new message Activity.
    await Dialog.Run(turnContext, ConversationState.CreateProperty<DialogState>(nameof(DialogState)), cancellationToken);
  }
}

{% endhighlight %}
</div>
Basically, this base class provides default implementation for <span class="code">OnTurnAsync</span> and <span class="code">OnMessageActivityAsync</span> to start a dialog with the user. </li><li>Now, let's create a new bot that inherits <span class="code">DialogBot</span> and duplicates <span class="code">OnMembersAddedAsync</span> logic of our initial bot. 
<div markdown="1">
{% highlight csharp %}

public class GraphBot<T> : DialogBot<T> where T : Dialog
{
  public GraphBot(ConversationState conversationState, UserState userState, T dialog, ILogger<DialogBot<T>> logger)
    : base(conversationState, userState, dialog, logger)
  {
  }

  protected override async Task OnMembersAddedAsync(IList<ChannelAccount> membersAdded, ITurnContext<IConversationUpdateActivity> turnContext, CancellationToken cancellationToken)
  {
    foreach (var member in membersAdded)
    {
      if (member.Id != turnContext.Activity.Recipient.Id)
      {
        await turnContext.SendActivityAsync(MessageFactory.Text($"Hello world!"), cancellationToken).ConfigureAwait(false);
      }
    }
  }
}

{% endhighlight %}
</div>
</li><li>Next - add <span class="code">MainDialog</span> implementation that will be an entry point Dialog of our bot, and move our Teams-related logic to it: 
<div markdown="1">
{% highlight csharp %}

public class MainDialog : ComponentDialog
{
  protected readonly ILogger _logger;

  public MainDialog(ILogger<MainDialog> logger) : base(nameof(MainDialog))
  {
    _logger = logger;

    AddDialog(new WaterfallDialog(nameof(WaterfallDialog), new WaterfallStep[]
    {
      DisplayContextInfoStepAsync
    }));

    InitialDialogId = nameof(WaterfallDialog);

  }

  private async Task<DialogTurnResult> DisplayContextInfoStepAsync(WaterfallStepContext stepContext, CancellationToken cancellationToken)
  {
    var teamsContext = stepContext.Context.TurnState.Get<ITeamsContext>();

    if (teamsContext != null) // the bot is used inside MS Teams
    {
      if (teamsContext.Team != null) // inside team
      {
        await stepContext.Context.SendActivityAsync(MessageFactory.Text($"Team Id: {teamsContext.Team.Id}"), cancellationToken).ConfigureAwait(false);
      }
      else // private or group chat
      {
        await stepContext.Context.SendActivityAsync(MessageFactory.Text($"We're in MS Teams but not in Team"), cancellationToken).ConfigureAwait(false);
      }
    }
    else // outside MS Teams
    {
      await stepContext.Context.SendActivityAsync(MessageFactory.Text("We're not in MS Teams context"), cancellationToken).ConfigureAwait(false);
    }

    return await stepContext.EndDialogAsync();
  }
}

{% endhighlight %}
</div>
We inherited the dialog from <span class="code">ComponentDialog</span>. It provides a strategy for creating independent dialogs to handle specific scenarios, breaking a large dialog set into more manageable pieces. Each of these pieces has its own dialog set, and avoids any name collisions with the dialog set that contains it. It's like a reusable component that aggregates multiple controls.<br />And if you look at the constructor code - we're adding <span class="code">WaterfallDialog</span> to internal dialog set. A <span class="code">WaterfallDialog</span> is a specific implementation of a dialog that is commonly used to collect information from the user or guide the user through a series of tasks. Steps of the dialog are executed sequentially. In our case right now there is the only step <span class="code">DisplayContextInfoStepAsync</span> to display Teams context info.<br />The last interesting part of the constructor is <span class="code">InitialDialogId = nameof(WaterfallDialog);</span>. We can have multiple dialogs inside <span class="code">ComponentDialog</span>, and this line of code shows the SDK what dialog to launch when <span class="code">MainDialog</span> is started. </li><li>Now, we can reference <span class="code">MainDialog</span> and new bot in <span class="code">Startup.cs</span> and delete old (initial) bot from the project.<br />Replace <span class="code">services.AddTransient&lt;IBot, EmptyBot&gt;();</span> with  
<div markdown="1">
{% highlight csharp %}

services.AddSingleton<MainDialog>();
services.AddTransient<IBot, GraphBot<MainDialog>>();

{% endhighlight %}
</div>
</li></ol>After doing all that we can test our bot in the simulator, deploy it to Azure and test from MS Teams. We should see the same results as before, but now - using Dialogs engine under the hood.<br /><br /><a name="auth"></a><h2>Authentication Time!</h2>Now when state and dialogs are configured, we're ready to implement authentication in the bot, and get access to MS Graph.<br />There are few different parts in this process as well.<br /><br /><h3>Register Additional Azure AD App for Authentication</h3>We've already registered one app while registering the bot... But, quoting <a href="https://docs.microsoft.com/en-us/azure/bot-service/bot-builder-authentication?view=azure-bot-service-4.0&tabs=aadv2%2Ccsharp%2Cbot-oauth#about-this-sample" target="_blank">official documentation</a>: <b>Whenever you register a bot in Azure, it gets assigned an Azure AD app. However, this app secures channel-to-bot access. You need an additional AAD app for each application that you want the bot to be able to authenticate on behalf of the user.</b><br />So, let's register additional app with needed permissions: <ol><li>Go to Azure AD -&gt; <b>App Registrations</b></li><li>Select <b>New registration</b></li><li>Enter name. In this sample - DEVTeamsGraphBotAuth</li><li>For account types select <b>Accounts in any organizational directory and personal Microsoft accounts (e.g. Skype, Xbox, Outlook.com)</b></li><li>Enter <span class="code">https://token.botframework.com/.auth/web/redirect</span> as Redirect URI<br /><img border="0" src="{{site.baseurl}}/assets/images/posts/2019/auth-azure-ad-app.png" /></li><li>Click <b>Register</b></li><li>Go to <b>Certificates &amp; secrets</b> and generate new Client Secret. Store both App ID and Client Secret for next steps.</li><li>Go to <b>API Permissions</b> and add permissions you're planning to use in the bot. In this sample we're going to access Group and Site resources. So the permissions are:<ul><li>Group.Read.All</li><li>Sites.Read.All</li></ul>We can also grant admin consent right away as <span class="code">Group.Read.All</span> permission requires admin consent. </li></ol><br /><h3>Update Bot Settings with Connection Setting</h3>Next step is to update Bot Settings in Azure - we need to provide a connection setting that reference newly registered Azure AD App. This configuration will be used to request authentication and permissions from the bot. <ol><li>Navigate to registered Bot Service</li><li>Go to <b>Settings</b> -&gt; <b>OAuth Connection Settings</b> -&gt; <b>Add Setting</b><br /><img border="0" src="{{site.baseurl}}/assets/images/posts/2019/bot-settings.png" /></li><li>Enter some name. For example, you can use the same name as for Azure AD App - DEVTeamsGraphBotAuth</li><li>For Service Provider select <b>Azure Active Directory v2</b></li><li>Enter stored Client ID and Client Secret of the Azure AD App</li><li>For Tenant ID you can either enter your Azure AD ID, or <b>common</b> to be available across tenants</li><li>For Scopes we need to enter the same permissions as for API Permissions section in Azure AD App: Group.Read.All and Sites.Read.All. The values should be separated by space<br /><img border="0" src="{{site.baseurl}}/assets/images/posts/2019/auth-connection-string.png" /></li><li>Hit <b>Save</b></li></ol>Now you can navigate back to the Connection Setting and click <b>Test Connection</b>. It will initiate the authentication process. And if it's successfull you'll see "Success" page like that:<br /><img border="0" src="{{site.baseurl}}/assets/images/posts/2019/test-token.png" /><br /><br /><h3>Add Authentication Step to the Bot</h3>Azure part is done. Now, let's do needed changes in the code to initiate authentication from there.<br />For that purposes we'll be using <span class="code">OAuthPrompt</span> dialog as a part of our <span class="code">MainDialog</span> flow. <ol><li>Add new property to <span class="code">appsettings.json</span> - <span class="code">ConnectionName</span>. Value of that property should be the name of the Connection Setting we've just added to the bot. In this sample - DEVTeamsGraphBotAuth</li><li>Modify <span class="code">MainDialog</span> constructor to have additional parameter - <span class="code">configuration</span>: 
<div markdown="1">
{% highlight csharp %}

public MainDialog(ILogger<MainDialog> logger, IConfiguration configuration) : base(nameof(MainDialog))

{% endhighlight %}
</div>
</li><li>Add <span class="code">OAuthPrompt</span> dialog in <span class="code">MainDialog</span> constructor: 
<div markdown="1">
{% highlight csharp %}

AddDialog(new OAuthPrompt(
  nameof(OAuthPrompt),
  new OAuthPromptSettings
  {
    ConnectionName = configuration["ConnectionName"],
    Text = "Please login",
    Title = "Login",
    Timeout = 300000
}));

{% endhighlight %}
</div>
</li><li>Add prompt step as the first step of the dialog to prompt for login: 
<div markdown="1">
{% highlight csharp %}

private async Task<DialogTurnResult> PromptStepAsync(WaterfallStepContext stepContext, CancellationToken cancellationToken)
{
  return await stepContext.BeginDialogAsync(nameof(OAuthPrompt), null, cancellationToken);
}

{% endhighlight %}
</div>
And in constructor: 
<div markdown="1">
{% highlight csharp %}

AddDialog(new WaterfallDialog(nameof(WaterfallDialog), new WaterfallStep[]
{
  PromptStepAsync,
  DisplayContextInfoStepAsync
}));

{% endhighlight %}
</div>
</li><li>Modify Info step to check for access token: 
<div markdown="1">
{% highlight csharp %}

private async Task<DialogTurnResult> DisplayContextInfoStepAsync(WaterfallStepContext stepContext, CancellationToken cancellationToken)
{
  if (stepContext.Result != null)
  {
    var tokenResponse = stepContext.Result as TokenResponse;
    if (tokenResponse?.Token != null)
    {
      // same code as before
    }
  }

  return await stepContext.EndDialogAsync();
}

{% endhighlight %}
</div>
The whole code of the <span class="code">MainDialog</span> class: 
<div markdown="1">
{% highlight csharp %}

public class MainDialog : ComponentDialog
{
  protected readonly ILogger _logger;

  public MainDialog(ILogger<MainDialog> logger, IConfiguration configuration) : base(nameof(MainDialog))
  {
    _logger = logger;

    AddDialog(new OAuthPrompt(
      nameof(OAuthPrompt),
      new OAuthPromptSettings
      {
        ConnectionName = configuration["ConnectionName"],
        Text = "Please login",
        Title = "Login",
        Timeout = 300000
      }));

    AddDialog(new WaterfallDialog(nameof(WaterfallDialog), new WaterfallStep[]
      {
        PromptStepAsync,
        DisplayContextInfoStepAsync
      }));

    InitialDialogId = nameof(WaterfallDialog);

  }

  private async Task<DialogTurnResult> PromptStepAsync(WaterfallStepContext stepContext, CancellationToken cancellationToken)
  {
    return await stepContext.BeginDialogAsync(nameof(OAuthPrompt), null, cancellationToken);
  }

  private async Task<DialogTurnResult> DisplayContextInfoStepAsync(WaterfallStepContext stepContext, CancellationToken cancellationToken)
  {
    if (stepContext.Result != null)
    {
      var tokenResponse = stepContext.Result as TokenResponse;

      if (tokenResponse?.Token != null)
      {
        var teamsContext = stepContext.Context.TurnState.Get<ITeamsContext>();

        if (teamsContext != null) // the bot is used inside MS Teams
        {
          if (teamsContext.Team != null) // inside team
          {
            await stepContext.Context.SendActivityAsync(MessageFactory.Text($"Team Id: {teamsContext.Team.Id}"), cancellationToken).ConfigureAwait(false);
          }
          else // private or group chat
          {
            await stepContext.Context.SendActivityAsync(MessageFactory.Text($"We're in MS Teams but not in Team"), cancellationToken).ConfigureAwait(false);
          }
        }
        else // outside MS Teams
        {
          await stepContext.Context.SendActivityAsync(MessageFactory.Text("We're not in MS Teams context"), cancellationToken).ConfigureAwait(false);
        }
      }
    }

    return await stepContext.EndDialogAsync();
  }
}

{% endhighlight %}
</div>
</li><li>Last thing to do is to add <span class="code">OnTokenResponseEventAsync</span> implementation to the bot itself (you can add it to the base <span class="code">DialogBot</span>). This event will be generated if the token has already been acquired before and there is no need for auth prompt: 
<div markdown="1">
{% highlight csharp %}

protected override async Task OnTokenResponseEventAsync(ITurnContext<IEventActivity> turnContext, CancellationToken cancellationToken)
{
  Logger.LogInformation("Running dialog with Token Response Event Activity.");

  // Run the Dialog with the new Token Response Event Activity.
  await Dialog.Run(turnContext, ConversationState.CreateProperty<DialogState>(nameof(DialogState)), cancellationToken);
}

{% endhighlight %}
</div>
</li></ol>Now we can publish the bot to the Azure and test it. It worth saying that Bot Emulator works weirdly with <span class="code">OAuthPrompt</span>, so it's better to test bot from Teams:<br /><img border="0" src="{{site.baseurl}}/assets/images/posts/2019/bot-login.png" /><br /><br /><a name="ms-graph"></a><h2>Add MS Graph Logic</h2>Let's finally add some logic to test that the authentication and Teams context work.<br />As mentioned before, we'll be getting Group based on the current Team, and some properties of the site associated with the group (team).<br /><br /><h3>Getting Group ID from Teams Context</h3>First step here is to get Group ID that can be used to request data from MS Graph.<br />If you look at <span class="code">TeamInfo</span> class which is a type of <span class="code">ITeamsContext.Team</span> property, you'll see that it contains Team Name and Id. Nothing else.<br />To get Group Id we need to perform additional operation included in <span class="code">ITeamsContext</span> - <span class="code">FetchTeamDetailsWithHttpMessagesAsync</span>.<br />Let's modify our code to get Team details if the conversation is happening inside a Team: 
<div markdown="1">
{% highlight csharp %}

if (teamsContext.Team != null) // inside team
{
  var team = teamsContext.Team;
  var teamDetails = await teamsContext.Operations.FetchTeamDetailsWithHttpMessagesAsync(team.Id);
       
  await stepContext.Context.SendActivityAsync(MessageFactory.Text($"Team Id: {teamsContext.Team.Id}"), cancellationToken).ConfigureAwait(false);
}

{% endhighlight %}
</div>
Now we have Group Id in <span class="code">teamDetails.Body.AadGroupId</span>. <br /><br /><h3>Requesting needed information from MS Graph</h3>Now, let's implement MS Graph part... <ol><li>Install <a href="https://www.nuget.org/packages/Microsoft.Graph/" target="_blank">Microsoft.Graph</a> NuGet package</li><li>Create Graph Client: 
<div markdown="1">
{% highlight csharp %}

var token = tokenResponse.Token;

var graphClient = new GraphServiceClient(
  new DelegateAuthenticationProvider(
  requestMessage =>
  {
    // Append the access token to the request.
    requestMessage.Headers.Authorization = new AuthenticationHeaderValue("bearer", token);

    // Get event times in the current time zone.
    requestMessage.Headers.Add("Prefer", "outlook.timezone=\"" + TimeZoneInfo.Local.Id + "\"");

    return Task.CompletedTask;
  })
);

{% endhighlight %}
</div>
</li><li>Get information about the site: 
<div markdown="1">
{% highlight csharp %}

var siteInfo = await graphClient.Groups[teamDetails.Body.AadGroupId].Sites["root"].Request().GetAsync();

{% endhighlight %}
</div>
</li><li>Display site properties: 
<div markdown="1">
{% highlight csharp %}

await stepContext.Context.SendActivityAsync(MessageFactory.Text($"Site Id: {siteInfo.Id}, Site Title: {siteInfo.DisplayName}, Site Url: {siteInfo.WebUrl}"), cancellationToken).ConfigureAwait(false);

{% endhighlight %}
</div>
</li></ol>The final code of <span class="code">DisplayContextInfoStepAsync</span> looks like that: 
<div markdown="1">
{% highlight csharp %}

private async Task<DialogTurnResult> DisplayContextInfoStepAsync(WaterfallStepContext stepContext, CancellationToken cancellationToken)
{
  if (stepContext.Result != null)
  {
    var tokenResponse = stepContext.Result as TokenResponse;
    if (tokenResponse?.Token != null)
    {
      var teamsContext = stepContext.Context.TurnState.Get<ITeamsContext>();

      if (teamsContext != null) // the bot is used inside MS Teams
      {
        if (teamsContext.Team != null) // inside team
        {
          var team = teamsContext.Team;
          var teamDetails = await teamsContext.Operations.FetchTeamDetailsWithHttpMessagesAsync(team.Id);
          var token = tokenResponse.Token;

          var graphClient = new GraphServiceClient(
            new DelegateAuthenticationProvider(
              requestMessage =>
              {
                // Append the access token to the request.
                requestMessage.Headers.Authorization = new AuthenticationHeaderValue("bearer", token);

                // Get event times in the current time zone.
                requestMessage.Headers.Add("Prefer", "outlook.timezone=\"" + TimeZoneInfo.Local.Id + "\"");

                return Task.CompletedTask;
            })
          );

          var siteInfo = await graphClient.Groups[teamDetails.Body.AadGroupId].Sites["root"].Request().GetAsync();

          await stepContext.Context.SendActivityAsync(MessageFactory.Text($"Site Id: {siteInfo.Id}, Site Title: {siteInfo.DisplayName}, Site Url: {siteInfo.WebUrl}"), cancellationToken).ConfigureAwait(false);
        }
        else // private or group chat
        {
          await stepContext.Context.SendActivityAsync(MessageFactory.Text($"We're in MS Teams but not in Team"), cancellationToken).ConfigureAwait(false);
        }
      }
      else // outside MS Teams
      {
        await stepContext.Context.SendActivityAsync(MessageFactory.Text("We're not in MS Teams context"), cancellationToken).ConfigureAwait(false);
      }
    }
  }

  return await stepContext.EndDialogAsync();
}

{% endhighlight %}
</div>
Now we can publish the code again and test in from a Team in Microsoft Teams. The result should be like that:<br /><img border="0" src="{{site.baseurl}}/assets/images/posts/2019/teams-bot-1.png" /><br /><br /><a name="next"></a><h2>Next Steps</h2>Now when we have the basement, we can improve our bot by adding different commands, routing between dialogs, displaying Adaptive Cards, etc.<br /><br /><a name="references"></a><h2>References</h2>Here are helpful links that were used to come up with this step-by-step guidance:<br /><ul><li><a href="https://marketplace.visualstudio.com/items?itemName=BotBuilder.botbuilderv4" target="_blank">https://marketplace.visualstudio.com/items?itemName=BotBuilder.botbuilderv4</a> - Visual Studio Templates for Bot Framework</li><li><a href="https://docs.microsoft.com/en-us/azure/bot-service/bot-builder-deploy-az-cli?view=azure-bot-service-4.0&tabs=newrg" target="_blank">https://docs.microsoft.com/en-us/azure/bot-service/bot-builder-deploy-az-cli?view=azure-bot-service-4.0&tabs=newrg</a> - deploying bot to Azure</li><li><a href="https://docs.microsoft.com/en-us/cli/azure/?view=azure-cli-latest" target="_blank">https://docs.microsoft.com/en-us/cli/azure/?view=azure-cli-latest</a> - Azure CLI</li><li><a href="https://github.com/Microsoft/BotFramework-Emulator/wiki/Getting-Started" target="_blank">https://github.com/Microsoft/BotFramework-Emulator/wiki/Getting-Started</a> - Bot Emulator Getting Started</li><li><a href="https://github.com/Microsoft/BotFramework-Emulator/wiki/Tunneling-(ngrok)" target="_blank">https://github.com/Microsoft/BotFramework-Emulator/wiki/Tunneling-(ngrok)</a> - Configuring Bot Emulator to use ngrok</li><li><a href="https://www.nuget.org/packages/Microsoft.Bot.Connector.Teams" target="_blank">https://www.nuget.org/packages/Microsoft.Bot.Connector.Teams</a> - Bot connector for Teams</li><li><a href="https://docs.microsoft.com/en-us/microsoftteams/platform/concepts/bots/bots-create" target="_blank">https://docs.microsoft.com/en-us/microsoftteams/platform/concepts/bots/bots-create</a> - creating bots for Teams</li><li><a href="https://github.com/OfficeDev/BotBuilder-MicrosoftTeams-dotnet" target="_blank">https://github.com/OfficeDev/BotBuilder-MicrosoftTeams-dotnet</a> - Bot Builder SDK 4 - Microsoft Teams Extensions</li><li><a href="https://mybuild.techcommunity.microsoft.com/sessions/77026?source=sessions#top-anchor" target="_blank">https://mybuild.techcommunity.microsoft.com/sessions/77026?source=sessions#top-anchor</a> - Jeremy Thake and Nick Kramer session around bots, teams and Graph from MS Build</li><li><a href="https://docs.microsoft.com/en-us/azure/bot-service/bot-builder-concept-state?view=azure-bot-service-4.0" target="_blank">https://docs.microsoft.com/en-us/azure/bot-service/bot-builder-concept-state?view=azure-bot-service-4.0</a> - Managing state in bots</li><li><a href="https://docs.microsoft.com/en-us/azure/bot-service/bot-builder-concept-dialog?view=azure-bot-service-4.0" target="_blank">https://docs.microsoft.com/en-us/azure/bot-service/bot-builder-concept-dialog?view=azure-bot-service-4.0</a> - Dialogs library</li><li><a href="https://github.com/microsoft/BotBuilder-Samples/tree/master/samples/csharp_dotnetcore/24.bot-authentication-msgraph" target="_blank">https://github.com/microsoft/BotBuilder-Samples/tree/master/samples/csharp_dotnetcore/24.bot-authentication-msgraph</a> - Bot authentication for MS Graph sample</li><li><a href="https://docs.microsoft.com/en-us/azure/bot-service/bot-builder-authentication?view=azure-bot-service-4.0&tabs=aadv2%2Ccsharp%2Cbot-oauth" target="_blank">https://docs.microsoft.com/en-us/azure/bot-service/bot-builder-authentication?view=azure-bot-service-4.0&tabs=aadv2%2Ccsharp%2Cbot-oauth</a> - Add authentication to your bot</li><li><a href="https://docs.microsoft.com/en-us/microsoftteams/platform/concepts/authentication/auth-flow-bot" target="_blank">https://docs.microsoft.com/en-us/microsoftteams/platform/concepts/authentication/auth-flow-bot</a> - Authentication flow for Bots in Teams</li><li><a href="https://github.com/microsoft/botframework-solutions" target="_blank">https://github.com/microsoft/botframework-solutions</a> - Bot Framework Soluitions - open source examples</li><li><a href="https://github.com/microsoftgraph/msgraph-sdk-dotnet" target="_blank">https://github.com/microsoftgraph/msgraph-sdk-dotnet</a> - MS Graph SDK for .NET</li><li><a href=" https://docs.microsoft.com/en-us/azure/bot-service/bot-service-manage-channels?view=azure-bot-service-4.0" target="_blank"> https://docs.microsoft.com/en-us/azure/bot-service/bot-service-manage-channels?view=azure-bot-service-4.0</a> - Connect a bot to channels</li></ul><br /><br /><a name="conclusion"></a><h2>Conclusion</h2>Connecting together bot, Microsoft Teams, and Microsoft Graph is not a rocket science. But it takes time and consists of pretty big number of steps, including configurations as well as coding.<br />Hopefully, this post will help to proceed with all the steps in correct order with ease.<br /><br />That's all for today!<br />Have fun!