---
layout: post
title: 'Outlook and SharePoint Events, Time Zones, Display Dates'
description: Overview of how Outlook and SharePoint events are displayed in different Microsoft 365 applications based on different Time Zones settings
date: 2020-10-21T22:55:13.586Z
tags:
  - Calendar
  - M365
  - MS Teams
  - Microsoft
  - Microsoft 365
  - Microsoft Teams
  - O365
  - Office 365
  - Outlook
  - SharePoint Online
  - Teams
  - Time Zone
  - Time Zones
  - tz
  - Regional Settings
  - Account
  - Profile
slug: outlook-sharepoint-events-time-zones-display-dates
featured_image: assets/images/posts/2020/2020-10-31-time-zones.jpg
featured_image_thumbnail: assets/images/posts/2020/2020-10-21-time-zones.jpg
---
Working with time zones is always difficult. Not only in SharePoint or Microsoft 365, but probably in any platform or application. In this post I want to overview how Outlook and SharePoint events dates are displayed in different Microsoft 365 applications based on different configurations available to the user.
<br />
<br />
<h2>SharePoint Events</h2>
Let's start with SharePoint events.<br />
There are 2 configurations that affect on how the event date/time is displayed to the user. Or, to be more specific, in what time zone the event date will be displayed.<br />
The first configuration is **site Regional Settings** that you can access in Site Settings -> Regional Settings:<br />
![Site Regional Settings](./assets/images/posts/2020/2020-10-21-site-regional-settings.png)<br />
In most cases, this is the only configuration set, and all users see events in the same time zone.<br />
But user can additionally set regional settings for himself. This is done in user's **Office Profile (Delve)**. [Here](https://support.microsoft.com/en-us/office/change-your-personal-language-and-region-settings-caa1fccc-bcdb-42f3-9e5b-45957647ffd7) you can find the documentation how to change the settings. Here is the screenshot of the documentation:<br />
![Office profile](./assets/images/posts/2020/2020-10-21-office-profile.png)<br />
If this configuration is set, a user will see the events dates in the same time zone on any SharePoint site.<br />
Good thing about SharePoint events - they will be displayed in the same time zone/format both in SharePoint and Microsoft Teams. So, the only possible confusion is where this time zone configuration comes from :)<br />
<h2>Outlook Events</h2> 
Now Outlook events.<br />
Outlook events are displayed based on the time zone set in user's **Microsoft Account** that can be accessed using [https://myaccount.microsoft.com](https://myaccount.microsoft.com) url.<br />
Single configuration, pretty easy, right?..<br />
But...<br />
First, Microsoft Account IS NOT EQUAL to Office Profile. It means if you have an app that synchronizes Outlook and SharePoint events, users can see different dates/times in SharePoint and Outlook based on their configurations in Microsoft Account and Office Profile.<br />
Second, if you look at the event in Microsoft Teams Calendar, you will see the event in the **current** time zone - the time zone of your machine, which can be confusing.
<h2>Conclusion</h2>
Below is an example for the event that is happening on October 21, 2020 5:00 PM UTC-8.<br />
My machine's time zone is UTC-8, Office Profile time zone is set to UTC, and Microsoft Account time zone is set to UTC+10:<br />
![Different configurations](./assets/images/posts/2020/2020-10-21-outlook-sp-teams.png)<br />
And here is a small summary table for the configurations and display results<br />
| Source  | Possible Configurations | Time Zone in SP | Time Zone in Outlook | Time Zone in Teams |
| ------- | ----------------------- | --------------- | -------------------- | ------------------ |
| Outlook | [Microsoft Account](https://myaccount.microsoft.com) | N/A | Set in Microsoft Account | Current machine's time zone |
| SharePoint | Site Regional Settings <br />[Office Profile](https://support.microsoft.com/en-us/office/change-your-personal-language-and-region-settings-caa1fccc-bcdb-42f3-9e5b-45957647ffd7) | Office Profile time zone, if set; Site time zone otherwise | N/A | Office Profile time zone, if set; Site time zone otherwise |
Hope, this information helps you and your users to resolve the confusions.

<br />
That's all for today!<br />
Have fun!