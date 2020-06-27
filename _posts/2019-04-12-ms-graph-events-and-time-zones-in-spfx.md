---
layout: post
title: MS Graph, Events, and Time Zones in SPFx
date: '2019-04-12T11:14:00.001-07:00'
author: Alex Terentiev
tags:
- MS Graph
- tz
- Time Zones
- Outlook
- SharePoint Framework
- Office 365
- IANA
- Intl
- Microsoft Graph
- SPFx
- Calendar
- Office Development
- Time Zone
- Microsoft Teams
- O365
modified_time: '2019-04-12T11:26:51.469-07:00'
thumbnail: https://3.bp.blogspot.com/-WsmPVhT1OrQ/XLDAdWz9YhI/AAAAAAAABFg/qqdvLBbIdEcitCowQnt3pTFMBSEiezhvwCLcBGAs/s72-c/Screen%2BShot%2B2019-04-12%2Bat%2B9.43.43%2BAM.png
blogger_id: tag:blogger.com,1999:blog-3066084330774405472.post-6698917691795540265
blogger_orig_url: http://blog.aterentiev.com/2019/04/ms-graph-events-and-time-zones-in-spfx.html
---

Working with time zones is always a mess, whatever language, technology stack or environment you're in. There are always different issues on how to convert dates, get correct time zone and so on.<br />Currently I'm working on SPFx solution that should create an event in Outlook calendar.<br />User should be able to select date, time, and time zone of the event.<br />Time zone selection dropdown should display friendly names like that:<br /><a href="https://3.bp.blogspot.com/-WsmPVhT1OrQ/XLDAdWz9YhI/AAAAAAAABFg/qqdvLBbIdEcitCowQnt3pTFMBSEiezhvwCLcBGAs/s1600/Screen%2BShot%2B2019-04-12%2Bat%2B9.43.43%2BAM.png" imageanchor="1"><img border="0" data-original-height="616" data-original-width="852" src="https://3.bp.blogspot.com/-WsmPVhT1OrQ/XLDAdWz9YhI/AAAAAAAABFg/qqdvLBbIdEcitCowQnt3pTFMBSEiezhvwCLcBGAs/s1600/Screen%2BShot%2B2019-04-12%2Bat%2B9.43.43%2BAM.png" width="600" /></a><br />And the dropdown initial selection should be based on browser's (computer) time zone.<br />Below I'm providing some thoughts on the implementation, unfortunately without final solution for now. All the thoughts are based on the SPFx stack. <br /><a name='more'></a><br /><h2>Implementation Steps</h2>So, what should be done? <ul><li>Create date time picker to select the date and time of the event.<br />This step is simple, and we can use <a href="https://github.com/SharePoint/sp-dev-fx-controls-react/" target="_blank">PnP Reusable Controls</a> for that </li><li>Create time zones dropdown and display friendly time zones names. <br />Not an issue as well. We can use <a href="https://docs.microsoft.com/en-us/graph/api/outlookuser-supportedtimezones" target="_blank"><span class="code">/me/outlook/supportedTimezones</span> MS Graph endpoint</a> to get all user's time zones: 
<div markdown="1">
{% highlight console %}

GET https://graph.microsoft.com/v1.0/me/outlook/supportedTimezones

{% endhighlight %}
</div>
The result will look like that:<br /><a href="https://3.bp.blogspot.com/-xZJxN9xaLTM/XLDIzci8eeI/AAAAAAAABF4/UZsolh7SB3MGjkxfQ3XV7hiBhGdzT2vnACLcBGAs/s1600/Screen%2BShot%2B2019-04-12%2Bat%2B10.19.58%2BAM.png" imageanchor="1" ><img border="0" src="https://3.bp.blogspot.com/-xZJxN9xaLTM/XLDIzci8eeI/AAAAAAAABF4/UZsolh7SB3MGjkxfQ3XV7hiBhGdzT2vnACLcBGAs/s1600/Screen%2BShot%2B2019-04-12%2Bat%2B10.19.58%2BAM.png" data-original-width="1600" data-original-height="687" width="600" /></a></li><li>Set time zones dropdown's selected item to current time zone.<br />This is a step with problems, and I will describe them below. </li><li>Create new event using data entered by a user.<br />Complicated step as well. But if we have current browser's time zone (and/or it's offset - which is always available), and new selected time zone with its offset - we can convert the date and time to needed time zone to send to the server using <a href="https://docs.microsoft.com/en-us/graph/api/user-post-events?view=graph-rest-1.0" target="_blank">Create Event method</a> of MS Graph: 
<div markdown="1">
{% highlight console %}

POST /me/events
{
  "subject": "Let's go for lunch",
  "body": {
    "contentType": "HTML",
    "content": "Does late morning work for you?"
  },
  "start": {
      "dateTime": "2017-04-15T12:00:00",
      "timeZone": "Pacific Standard Time"
  },
  "end": {
      "dateTime": "2017-04-15T14:00:00",
      "timeZone": "Pacific Standard Time"
  },
  "location":{
      "displayName":"Harry's Bar"
  },
  "attendees": [
    {
      "emailAddress": {
        "address":"samanthab@contoso.onmicrosoft.com",
        "name": "Samantha Booth"
      },
      "type": "required"
    }
  ]
}

{% endhighlight %}
</div>
</li></ul>Below I want to describe all the problems that I experienced while working on the project. Once again, I don't have "good" final solution as of now.<br /><h2>Problem #1 - IE 11</h2>Browsers contain <span class="code">Intl</span> APIs that allow you to get current time zone: 
<div markdown="1">
{% highlight typescript %}

const timeZone: string = Intl.DateTimeFormat().resolvedOptions().timeZone;

{% endhighlight %}
</div>
It will return a string like <span class="code">America/Los_Angeles</span>, which is an Internet Assigned Numbers Authority time zone database code, or IANA code (<a href="https://www.iana.org/time-zones" target="_blank">source</a>, <a href="https://en.wikipedia.org/wiki/List_of_tz_database_time_zones" target="_blank">Wikipedia</a>).<br />It allows you to correctly define user's time zone and even location (more or less). The problem with it is you need to map it with user-friendly display name that is added to time zones dropdown. See <b>Problem #2 - IANA-Windows Time Zones Mapping</b>.<br />Everything is good, but <b>in IE 11 <span class="code">Intl.DateTimeFormat().resolvedOptions().timeZone</span> is <span class="code">undefined</span></b>...<br />OK, in that case we can get time zone offset from a <span class="code">Date</span> object: 
<div markdown="1">
{% highlight typescript %}

new Date().getTimezoneOffset()

{% endhighlight %}
</div>
It will give us the offset in minutes (for example, my current time zone is Pacific Standard with daylight offset and I will get 420 as a result). The problem with that approach is there is no information about location, and, as a result, we'll lose information if it's really a Pacific Daylight, or Arizona time zone (Arizona is UTC-7 without daylight saving).<br />Another option here is to get time zone setting from user's Outlook Calendar. It might differ from current time zone while traveling, but in most cases will be accurate.<br />We can do that using user's <a href="https://docs.microsoft.com/en-us/graph/api/resources/mailboxsettings?view=graph-rest-1.0" target="_blank">mailboxSettings resource</a> from MS Graph: 
<div markdown="1">
{% highlight console %}

GET https://graph.microsoft.com/v1.0/me/mailboxSettings

{% endhighlight %}
</div>
There response will contain <span class="code">timeZone</span> property with a string like "Pacific Standard Time" that can be used to initialize selected item of time zones dropdown. No need for additional mapping.<br /><h2>Problem #2 - IANA-Windows Time Zones Mapping</h2>This problem is related to the fact that in time zones dropdown we want to show user-friendly time zones (called Windows time zones), and browser's time zone is stored as IANA time zone code. And we need to map these two to select correct item in the dropdown based on browser's time zone.<br />Unfortunately, there is no one-to-one mapping between Windows and IANA time zones: one Windows time zone contains multiple IANA time zones.<br />Another problem here is that time zones and daylight saving rules are changed relevantly often, so, if you don't want to support your code forever, you don't want to hardcode the mapping.<br />First source I was looking into to solve the mapping issue was MS Graph. Mentioned earlier <span class="code">/me/outlook/supportedTimezones</span> endpoint allows us to get time zones in Windows format, as well as in IANA format, if we add <span class="code">TimeZoneStandard=microsoft.graph.timeZoneStandard'Iana'</span> parameter: 
<div markdown="1">
{% highlight console %}

POST https://graph.microsoft.com/v1.0/me/outlook/supportedTimezones(TimeZoneStandard=microsoft.graph.timeZoneStandard'Iana')

{% endhighlight %}
</div>
The problem here is that both responses that they return a list of <a href="https://docs.microsoft.com/en-us/graph/api/resources/timezoneinformation?view=graph-rest-1.0" target="_blank"><span class="code">timeZoneInformation</span> entities</a>. And these entities contain <span class="code">alias</span> and <span class="code">displayName</span> properties only. No offset or some other parameter that could be used to map items from these 2 lists.<br />For example, for Pacific Standard Time in Windows list I'll get 
<div markdown="1">
{% highlight javascript %}

{
  "alias": "Pacific Standard Time",
  "displayName": "(UTC-08:00) Pacific Time (US & Canada)"
}

{% endhighlight %}
</div>
And for IANA list I'll get 
<div markdown="1">
{% highlight javascript %}

{
  "alias": "America/Los_Angeles",
  "displayName": "America/Los_Angeles"
},
{
 "alias": "PST8PDT",
 "displayName": "PST8PDT"
},
{
 "alias": "Canada/Pacific",
 "displayName": "Canada/Pacific"
}

{% endhighlight %}
</div>
... and some others.<br />As you can see there is no way to map these values one to another.<br /><br /><b>NOTE:</b> I've posted a <a href="https://microsoftgraph.uservoice.com/forums/920506-microsoft-graph-feature-requests/suggestions/37360369-provide-mapping-between-windows-and-iana-time-zone" target="_blank">MS Graph UserVoice idea</a> to provide some kind of mapping in the results. So, if you feel like this will solve your pain, please, go and vote for it. <br /><br />So, after failure with MS Graph I was looking for some existing <b>free</b> APIs/endpoints to get this mapping. And, unfortunately, I was unable to find any... Remark here is that I was looking for commercial usage, there are some free resources for personal usage, but not for commercial.<br />The only more or less interesting resource I found is <a href="http://unicode.org/repos/cldr/trunk/common/supplemental/windowsZones.xml" target="_blank">windowsZones.xml</a> hosted by Unicode.org. They update it more or less regularly, so it can be used to get the mapping.<br />The problem here is that while implementing logic to get that file (with caching of course) from the url above, parse it and return in needed format, I experienced 503s multiple times...<br />So, I decided, that I can't rely on that. <br /><br />As of now, the only solution here from my point of view is  <ul><li>Get either this XML, or full IANA database, and copy it to your custom location</li><li>Implement REST API (or any other API) to get mapping in needed format</li><li><b>Worst part:</b> do not forget to update the file/database if a new version was released</li></ul>It's not so bad... But if we could have this mapping in MS Graph... :)<br /><h2>Conclusion</h2>Time zones conversion itself is difficult in implementation. But this part is mostly solved with such libraries as <a href="https://momentjs.com/timezone/" target="_blank">moment-timezones</a>.<br />Unfortunately, there are other problems that we can face while working on projects with dates.<br />Two of them are listed above: IE 11 doesn't show current time zone info, and there is no reliable easy-to-support solution for Windows-IANA time zones mapping.<br />Please, vote for <a href="https://microsoftgraph.uservoice.com/forums/920506-microsoft-graph-feature-requests/suggestions/37360369-provide-mapping-between-windows-and-iana-time-zone" target="_blank">MS Graph UserVoice idea</a> to solve at least the second problem with out-of-the-box Office 365 implementation.<br /><br />That's all for today!<br />Have fun!