---
layout: post
title: Getting Related Team Information from Private Channel in SPFx MS Teams App
date: '2019-11-20T13:54:00.000-08:00'
author: Alex Terentiev
tags:
- MS Graph
- Private Channel
- SharePoint Framework
- Office 365
- SharePoint Development
- SharePoint
- Client Side Web Part
- SPFx
- Office Development
- SPFx Web Parts
- Microsoft Teams
- MS Teams
- O365
- Private Channels
modified_time: '2019-11-20T13:54:04.946-08:00'
blogger_id: tag:blogger.com,1999:blog-3066084330774405472.post-1840473478477100715
blogger_orig_url: http://blog.aterentiev.com/2019/11/getting-related-team-information-from.html
---

Just two weeks ago during <a href="https://myignite.techcommunity.microsoft.com/sessions" target="_blank">MS Ignite</a> Microsoft Teams team has announced <a href="https://docs.microsoft.com/en-us/microsoftteams/private-channels" target="_blank">Private channels</a> - one of the most asked features.<br />So, a bit of technical details. Each private channel is a separate SharePoint site collection with a unique site template and no group assigned. <br />We can discuss a lot if it's good decision or not.<br />But in this post I want to focus on a specific problem I experienced while developing SPFx solution for MS Teams. <br /><a name='more'></a><h2>The Problem</h2>You are developing a SharePoint Framework solution to be served as a Teams Tab. And you want to display some basic team (and maybe group) information in your app.<br />In general scenario everything looks pretty straightforward: you're getting team context as described <a href="https://docs.microsoft.com/en-us/sharepoint/dev/spfx/web-parts/get-started/using-web-part-as-ms-teams-tab#updating-code-to-be-aware-of-the-microsoft-teams-context" target="_blank">here</a> and using <span class="code">groupId</span> property and MS Graph you now can get any information about team or group you want.<br />But, surprise-surprise! This won't work for Private Channels!<br />Well, you'll still be able to get the context, but <span class="code">groupId</span> is <span class="code">undefined</span>. Moreover, <span class="code">teamName, teamId, teamSite</span> are actually name, id and site collection url of that private channel. So, <b>there is no any team-related information in team's context</b>.<br />If you look at web part's <span class="code">pageContext</span> - it's all about private channel's site collection as well...<br />At that point you can feel really stuck and maybe angry :)<br />But I have good news for you - there is a solution<br /><h2>The Solution</h2>You would expect that private channel's site collection somehow "knows" about related team and group...<br />And that is actually true: there is a <span class="code">RelatedGroupId</span> property in the site collection object as well as in root web's property bag. And it contains the needed <span class="code">groupId</span> value of the related team. So, we can use SharePoint REST API to get that value: 
<div markdown="1">
{% highlight javascript %}

https://contoso.sharepoint.com/sites/private-site-collection-url/_api/site?$select=RelatedGroupId
// or
https://contoso.sharepoint.com/sites/private-site-collection-url/_api/web/allProperties

{% endhighlight %}
</div>
One small remark here: if you the site collection API (the first one) and don't specify the <span class="code">$select</span> part you <b>will not</b> get that property back.<br />OK, now we can integrate that in our web part: 
<div markdown="1">
{% highlight typescript %}

const siteResponse = await this.context.spHttpClient.get(`${this.context.pageContext.site.absoluteUrl}/_api/site?$select=RelatedGroupId`, SPHttpClient.configurations.v1);
const siteProps = await siteResponse.json();
const groupId: string = siteProps.RelatedGroupId;

const graphClient = await this.context.msGraphClientFactory.getClient();
const team = await graphClient.api(`/teams/${groupId}`).version('v1.0').get();

// do something with the team


{% endhighlight %}
</div>
And, most probably, you don't want to do additional requests if you're not in a private channel.<br />To check that we can use new team's context property <span class="code">channelType</span>. It is equal to <span class="code">Private</span> for private channels.<br />One thing about this property is it's not exposed to <span class="code">microsoftTeams.Context</span> interface that is shipped with SPFx v 1.9.1 (the latest one). So, you'll need to do something like <span class="code">teamsContext as any</span> to use it: 
<div markdown="1">
{% highlight typescript %}

let groupId = this._teamsContext.groupId;
const teamsContext = this._teamsContext as any;
if (teamsContext.channelType === 'Private') {
  const siteResponse = await this.context.spHttpClient.get(`${this.context.pageContext.site.absoluteUrl}/_api/site?$select=RelatedGroupId`, SPHttpClient.configurations.v1);
  const siteProps = await siteResponse.json();
  const groupId: string = siteProps.RelatedGroupId;
}

const graphClient = await this.context.msGraphClientFactory.getClient();
const team = await graphClient.api(`/teams/${groupId}`).version('v1.0').get();

// do something with the team

{% endhighlight %}
</div>
<br />Hopefully, I'll save you some time with that post!<br /><br />That's all for today!<br />Have fun!