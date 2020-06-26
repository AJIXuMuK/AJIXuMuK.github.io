---
layout: post
title: Support Microsoft Teams Themes in SharePoint Framework Solutions
date: '2020-03-28T15:26:00.000-07:00'
author: Alex Terentiev
tags:
- SharePoint Online
- SCSS
- Microsoft 365
- SharePoint Framework
- SharePoint
- CSS
- SPFx
- SPFx Web Parts
- Microsoft Teams
- MS Teams
- Themes
- Dark Theme
modified_time: '2020-03-29T12:33:04.938-07:00'
thumbnail: https://1.bp.blogspot.com/-EjL9kCrCcpM/Xn6M8_KoliI/AAAAAAAABj8/XV-E0e98grAT1t0wJbXqNSw1h6aPPyHxQCLcBGAsYHQ/s72-c/Screen%2BShot%2B2020-03-27%2Bat%2B4.31.40%2BPM.png
blogger_id: tag:blogger.com,1999:blog-3066084330774405472.post-696348623156838468
blogger_orig_url: http://blog.aterentiev.com/2020/03/support-microsoft-teams-themes-in.html
---
<style>
    .code {
        font-family: Consolas, "courier new", "courier", monospace;
        font-size: 13px;
    }
</style>
If you are a SharePoint Framework developer, you're most likely aware that SPFx allows you to refer to the theme
colors of the context site. As a result, if your web part is placed on a site that uses a red theme, it uses the red
palette as well, and if it's placed on a site that uses the blue theme, it automatically adjusts itself to use the blue
palette. All of this is done automatically without any changes to the web part code in between.<br />But what if you
expose your web part to Microsoft Teams? And selected theme in Teams is Dark or Contrast?<br />Unfortunately, SharePoint
Framework doesn't handle MS Teams themes for you, you should do that by yourself.<br />And it's not a big deal if you
have few controls in the web part.<br />But what if your project contains tens of different components?<br />In this
post I want to share the approach I used to support MS Teams themes in the web part that has about 50 different
components. <b>Note:</b> I use React for production SPFx development, so all the thoughts below may not be applicable to
other frameworks, especially if the components are not so encapsulated as in React. <br /><a name='more'></a><br />
<h2>Initial State</h2>Let's say, you were developing your web part for SharePoint. And as a good developer, you have a
separate folder with a separate CSS (SCSS) for each component. And, of course, you use SPFx theme colors.<br />As a
result, you have a project structure like:<br /><a
    href="https://1.bp.blogspot.com/-EjL9kCrCcpM/Xn6M8_KoliI/AAAAAAAABj8/XV-E0e98grAT1t0wJbXqNSw1h6aPPyHxQCLcBGAsYHQ/s1600/Screen%2BShot%2B2020-03-27%2Bat%2B4.31.40%2BPM.png"
    imageanchor="1"><img border="0"
        src="https://1.bp.blogspot.com/-EjL9kCrCcpM/Xn6M8_KoliI/AAAAAAAABj8/XV-E0e98grAT1t0wJbXqNSw1h6aPPyHxQCLcBGAsYHQ/s1600/Screen%2BShot%2B2020-03-27%2Bat%2B4.31.40%2BPM.png"
        data-original-width="618" data-original-height="620" width="700" /></a><br />And inside your SCSS you can have
something like:
<div markdown="1">
{% highlight css %}
    .firstComponent {
        background: "[theme:white, default:#fff]";
        color: "[theme:primaryText, default:#333]";
        
        .button {
            background: "[theme:themePrimary, default:#0078d4]";
            color: "[theme:white, default:#fff]";
        }
    }
{% endhighlight %}
</div>
And it will look always good in SharePoint:<br /><a
    href="https://2.bp.blogspot.com/-AidCxv_l1Zo/Xn6RppK8hOI/AAAAAAAABkU/-xXdtkKihqoSdGQz_KHz5byPD-8CGsVdQCLcBGAsYHQ/s1600/Screen%2BShot%2B2020-03-27%2Bat%2B4.51.37%2BPM.png"
    imageanchor="1"><img border="0"
        src="https://2.bp.blogspot.com/-AidCxv_l1Zo/Xn6RppK8hOI/AAAAAAAABkU/-xXdtkKihqoSdGQz_KHz5byPD-8CGsVdQCLcBGAsYHQ/s1600/Screen%2BShot%2B2020-03-27%2Bat%2B4.51.37%2BPM.png"
        data-original-width="1600" data-original-height="505" width="700" /></a><br />But in MS Teams, especially with
dark theme, it may look not so good:<br /><a
    href="https://1.bp.blogspot.com/-XGr7mesYRE4/Xn6SAMF13EI/AAAAAAAABkc/nLFjStdvDFw-h2JOr858UZtQoNjyS8LqACLcBGAsYHQ/s1600/Screen%2BShot%2B2020-03-27%2Bat%2B4.53.24%2BPM.png"
    imageanchor="1"><img border="0"
        src="https://1.bp.blogspot.com/-XGr7mesYRE4/Xn6SAMF13EI/AAAAAAAABkc/nLFjStdvDFw-h2JOr858UZtQoNjyS8LqACLcBGAsYHQ/s1600/Screen%2BShot%2B2020-03-27%2Bat%2B4.53.24%2BPM.png"
        data-original-width="1600" data-original-height="807" width="700" /></a><br />It happens because your web part
is still "lives" in the context of the underlying site. And, in my case, it has "green" theme.<br />And if you use
Office UI Fabric components, all of them will either use their own default styles or the theme from SharePoint site as
well:<br /><a
    href="https://2.bp.blogspot.com/-5Va-I-IvZJg/Xn6TMCDCfmI/AAAAAAAABko/2ZuangFrg7sthFZcKI6UJG3jouEhfkVeACLcBGAsYHQ/s1600/Screen%2BShot%2B2020-03-27%2Bat%2B4.58.15%2BPM.png"
    imageanchor="1"><img border="0"
        src="https://2.bp.blogspot.com/-5Va-I-IvZJg/Xn6TMCDCfmI/AAAAAAAABko/2ZuangFrg7sthFZcKI6UJG3jouEhfkVeACLcBGAsYHQ/s1600/Screen%2BShot%2B2020-03-27%2Bat%2B4.58.15%2BPM.png"
        data-original-width="1600" data-original-height="652" width="700" /></a><br />So, to handle Teams themes
correctly we'll need:<br />
<ul>
    <li>Handle our own components' styles</li>
    <li>Correctly override Office UI Fabric styles</li>
</ul>Let's see how we can achieve that.<br /><br />
<h2>1. Handle Theme change in MS Teams</h2>The first step is to handle theme change in Teams.<br />And Teams JavaScript
SDK contains a handler <span class="code">registerOnThemeChangeHandler</span> that we can use to be informed when the
theme is changed.<br />Team's <span class="code">context</span> also contains <span class="code">theme</span> property
that shows the current theme: default, dark, or contrast.<br />Knowing that, we can use the next code to handle the
theming:
<div markdown="1">
{% highlight ts %}
protected async onInit(): Promise&lt;void&gt; {
    if (this.context.sdks.microsoftTeams) { 
        // checking that we're in Teams
        const context = this.context.sdks.microsoftTeams!.context;
        this._applyTheme(context.theme || 'default');
        this.context.sdks.microsoftTeams.teamsJs.registerOnThemeChangeHandler(this._applyTheme);
    }
}
private _applyTheme = (theme: string): void =&gt; {
    this.context.domElement.setAttribute('data-theme', theme);
    document.body.setAttribute('data-theme', theme);
    {% endhighlight %}
</div>
So, during the initialization, and whenever the theme is changed we are setting <span class="code">data-theme</span>
attribute of document's body to the selected <span class="code">theme</span>. <h3>Why data attribute on body?</h3>You
may ask why are we setting data attribute? And why on body, not the web part's root DOM element?<br />There are multiple
reasons for that: <ul>
    <li>All "layer" components of Office UI Fabric (Dialogs, Panels, etc.) are rendered outside of the web part's DOM.
        If our attribute is set on the web part's root DOM element we won't be able to process these "layer" components.
    </li>
    <li>We can't use scoped CSS class names (className_&lt;hash&gt;). The reason for that is the styles are scope for
        each component separately.<br />And if we assign, for example, <span class="code">styles.dark</span> class in
        the web part's root component it will be trasformed to something like <span
            class="code">dark_6e6e1386</span>.<br />But in our <span class="code">FirstComponent</span> it will be <span
            class="code">dark_32a77d9d</span>.<br />As a result we won't be able to use nested CSS rules like:
        <pre class="brush: css;"><br />.dark {<br />  .firstComponent {<br />  }<br />}<br /></pre>
    </li>
    <li><b>You can use global class name instead of data attribute</b>. I just prefer data attribute.</li>
</ul><br />
<h2>2. Design Your Components for Teams Themes</h2>Next step is to select what colors to use for each of 3 Teams
themes.<br />In ideal world designer should help you with that...<br />But in real world we can use some helper tools to
achieve it by ourselves.<br />For example, we can use <a href="https://aka.ms/spthemebuilder" target="_blank">SharePoint
    Theme Generator</a>. Good thing in using it - it will generate a set of the same variables that we used for
SharePoint. For example, you'll have white, primaryText and themePrimary colors. And as a result, you could use these
generated colors in the same place where the default variable was used.<br />So, for default Teams theme let's set the
next values in the Generator:<br />
<ul>
    <li><b>Primary theme color:</b> #6264a7</li>
    <li><b>Body text color:</b> #252423</li>
    <li><b>Body background color:</b> #F3F2F1</li>
</ul>And now in the Designer we have all the theme variables assigned:<br /><a
    href="https://2.bp.blogspot.com/-7fmMbaQlorU/Xn6a-wH2tJI/AAAAAAAABk0/ldwlWArdeuQV0wKwecIQmVEcAGY9uzU0gCLcBGAsYHQ/s1600/Screen%2BShot%2B2020-03-27%2Bat%2B5.31.36%2BPM.png"
    imageanchor="1"><img border="0"
        src="https://2.bp.blogspot.com/-7fmMbaQlorU/Xn6a-wH2tJI/AAAAAAAABk0/ldwlWArdeuQV0wKwecIQmVEcAGY9uzU0gCLcBGAsYHQ/s1600/Screen%2BShot%2B2020-03-27%2Bat%2B5.31.36%2BPM.png"
        data-original-width="1600" data-original-height="1399" width="700" /></a><br />Store generated variables for
future use.<br />And let's do the same for dark and contrast.<br /><br />Dark: <ul>
    <li><b>Primary theme color:</b> #6264a7</li>
    <li><b>Body text color:</b> #ffffff</li>
    <li><b>Body background color:</b> #2d2c2c</li>
</ul><br />Contrast: <ul>
    <li><b>Primary theme color:</b> #6264a7</li>
    <li><b>Body text color:</b> #ffffff</li>
    <li><b>Body background color:</b> #0000000</li>
</ul><br />Of course, SharePoint Theme Generator can't give us the ideal result. Especially for contrast theme. And it's
still recommended to either tweak the colors a bit if needed or beg a designer to help you :)<br /><br />
<h2>3. Define Variables for Each Color and Each Theme</h2>Next step is to create SASS variables.<br />For example,
instead of direct
<pre class="brush: css;"><br />.firstComponent {<br />  background: "[theme:white, default:#fff]";<br />}<br /></pre>We
can use variable:
<pre
    class="brush: css;"><br />$background: "[theme:white, default:#fff]";<br />.firstComponent {<br />  background: $background;<br />}<br /></pre>
Moreover, these variables can be define in a separate scss file and shared between different components.<br />I would
also recommend to provide pretty specific names for the variables. For example, if you want to use some color as <span
    class="code">FirstComponent</span> background, name the variable <span
    class="code">$firstComponent-background</span>.<br />Let's create variables for all our custom color:
<pre
    class="brush: typescript;"><br />//SharePoint<br />$firstComponent-background: "[theme:white, default:#fff]";<br />$firstComponent-color: "[theme:primaryText, default:#333]";<br />$firstComponentButton-background: "[theme:themePrimary, default:#0078d4]";<br />$firstComponentButton-color: "[theme:white, default:#fff]";<br /><br />// default theme<br />$default-firstComponent-background: #f3f2f1;<br />$default-firstComponent-color: #252423;<br />$default-firstComponentButton-background: #6264a7;<br />$default-firstComponentButton-color: #f3f2f1;<br /><br />// dark theme<br />$dark-firstComponent-background: #2d2c2c;<br />$dark-firstComponent-color: #ffffff;<br />$dark-firstComponentButton-background: #6264a7;<br />$dark-firstComponentButton-color: #2d2c2c;<br /><br />// contrast theme<br />$contrast-firstComponent-background: #000000;<br />$contrast-firstComponent-color: #ffffff;<br />$contrast-firstComponentButton-background: #6264a7;<br />$contrast-firstComponentButton-color: #000000;<br /></pre>
And let's define all these variables in the separate module <span class="code">_colors.module.scss</span> in <span
    class="code">common</span>.<br /><a
    href="https://1.bp.blogspot.com/-N1XMSa5QXXg/Xn6iMaQJ3VI/AAAAAAAABlA/x83dai7OtlYqAFtAK-ZXQHGWKO_AWPzwACLcBGAsYHQ/s1600/Screen%2BShot%2B2020-03-27%2Bat%2B6.02.23%2BPM.png"
    imageanchor="1"><img border="0"
        src="https://1.bp.blogspot.com/-N1XMSa5QXXg/Xn6iMaQJ3VI/AAAAAAAABlA/x83dai7OtlYqAFtAK-ZXQHGWKO_AWPzwACLcBGAsYHQ/s1600/Screen%2BShot%2B2020-03-27%2Bat%2B6.02.23%2BPM.png"
        data-original-width="612" data-original-height="312" width="700" /></a><br />Now we can reference this file in
any of our components. <br /><br />
<h2>4. Override Styles for Your Custom Components</h2>Now, when we have global data attribute set and variables for all
the themes we can override styles for our custom components for each theme. And they will be automatically applied as
this or that value of data attribute is set.<br />Again, using example of the <span class="code">FirstComponent</span>
we'll have:
<pre
    class="brush: css;"><br />@import "../../common/colors.module";<br /><br />.firstComponent {<br />  background: "[theme:white, default:#fff]";<br />  color: "[theme:primaryText, default:#333]";<br /><br />  .button {<br />    background: "[theme:themePrimary, default:#0078d4]";<br />    color: "[theme:white, default:#fff]";<br />}<br />}<br /><br />[data-theme='default'] {<br />  .firstComponent {<br />    background: $default-firstComponent-background;<br />    color: $default-firstComponent-color;<br /><br />    .button {<br />      background: $default-firstComponentButton-background;<br />      color: $default-firstComponentButton-color;<br />    }<br />  }<br />}<br /><br />[data-theme='dark'] {<br />  .firstComponent {<br />    background: $dark-firstComponent-background;<br />    color: $dark-firstComponent-color;<br /><br />    .button {<br />      background: $dark-firstComponentButton-background;<br />      color: $dark-firstComponentButton-color;<br />    }<br />  }<br />}<br /><br />[data-theme='contrast'] {<br />  .firstComponent {<br />    background: $contrast-firstComponent-background;<br />    color: $contrast-firstComponent-color;<br /><br />    .button {<br />      background: $contrast-firstComponentButton-background;<br />      color: $contrast-firstComponentButton-color;<br />    }<br />  }<br />}<br /></pre>
So, here we still use site theme if the web part is rendered in SharePoint. But we also have different colors for
different themes in Microsoft Teams.<br />And now our component looks much better in Teams:<br /><a
    href="https://2.bp.blogspot.com/-Z1it88Zctp4/Xn-6gRRKTjI/AAAAAAAABlM/NMUNm5grOgYz8BM54lUKmcS4TjSWHEDjQCLcBGAsYHQ/s1600/Screen%2BShot%2B2020-03-28%2Bat%2B1.55.02%2BPM.png"
    imageanchor="1"><img border="0"
        src="https://2.bp.blogspot.com/-Z1it88Zctp4/Xn-6gRRKTjI/AAAAAAAABlM/NMUNm5grOgYz8BM54lUKmcS4TjSWHEDjQCLcBGAsYHQ/s1600/Screen%2BShot%2B2020-03-28%2Bat%2B1.55.02%2BPM.png"
        data-original-width="1020" data-original-height="524" width="700" /></a><br /><a
    href="https://1.bp.blogspot.com/-2Cb1BsogpXE/Xn-6nNnQU2I/AAAAAAAABlQ/HkmiPHo5KwgLP0orxI6I-ns60hm5IAV3ACLcBGAsYHQ/s1600/Screen%2BShot%2B2020-03-28%2Bat%2B1.54.47%2BPM.png"
    imageanchor="1"><img border="0"
        src="https://1.bp.blogspot.com/-2Cb1BsogpXE/Xn-6nNnQU2I/AAAAAAAABlQ/HkmiPHo5KwgLP0orxI6I-ns60hm5IAV3ACLcBGAsYHQ/s1600/Screen%2BShot%2B2020-03-28%2Bat%2B1.54.47%2BPM.png"
        data-original-width="1392" data-original-height="618" width="700" /></a><br /><br />
<h2>5. Override Global Office UI Fabric Styles</h2>So, custom components now look good in Teams. But if you use Office
UI Fabric (OUIFR) components - they still don't respect Teams themes.<br /><a
    href="https://3.bp.blogspot.com/-3egOsjGiFfk/Xn-7YCdnVuI/AAAAAAAABlg/DMtXQYsyWl0S4jzRvYmvw1BoJUIUNWfcQCLcBGAsYHQ/s1600/Screen%2BShot%2B2020-03-28%2Bat%2B2.02.04%2BPM.png"
    imageanchor="1"><img border="0"
        src="https://3.bp.blogspot.com/-3egOsjGiFfk/Xn-7YCdnVuI/AAAAAAAABlg/DMtXQYsyWl0S4jzRvYmvw1BoJUIUNWfcQCLcBGAsYHQ/s1600/Screen%2BShot%2B2020-03-28%2Bat%2B2.02.04%2BPM.png"
        data-original-width="1600" data-original-height="754" width="700" /></a><br />To override all the styles
correctly, you'll need to figure out what OUIFR components are used and what classes they have.<br />For example, we use
<span class="code">Panel</span> component. This component use such classes as <span class="code">ms-Layer, ms-Panel,
    ms-Overlay</span> and so on.<br />Next step is to analyze DOM element with which class sets applies color styles
(backgrounds, borders, font colors, shadows, etc.)<br />Doing that for the <span class="code">Panel</span> in our
example we'll figure out that we need to override: <ul>
    <li><span class="code">.ms-Fabric</span></li>
    <li><span class="code">.ms-Button-icon</span></li>
    <li><span class="code">.ms-Overlay</span></li>
    <li><span class="code">.ms-Panel-main</span></li>
    <li><span class="code">.ms-Panel-headerText</span></li>
</ul>You can do all the global overrides in the root component's CSS, but to make it more manageable for large projects
I would recommend to create 3 separate files in <span class="code">common</span> folder for all 3 themes and import them
in the root component.<br />But first, let's add variables that will be used in the overrides into <span
    class="code">_colors.module.scss</span>. Again, you can get most of the colors from SharePoint Theme Generator. If
some values do exist in <span class="code">window.__themeState__.theme</span> but not in the Generator then just switch
SharePoint site theme to the one that is close enough to Teams theme and get values from there.<br />
<pre
    class="brush: typescript;"><br />//SharePoint<br />$overlay: "[theme:whiteTranslucent40, default:rgba(255, 255,255, 0.4)]";<br />$surfaceBackground: "[theme:white, default:#fff]";<br />$primaryText: "[theme:primaryText, default:#333]";<br />$panelBorder: "[theme: neutralLight, default: #eaeaea]";<br /><br />// default theme<br />$default-overlay: rgba(255, 255, 255, 0.4);<br />$default-surfaceBackground: #f3f2f1;<br />$default-primaryText: #252423;<br />$default-panelBorder: #dedddc;<br /><br />// dark theme<br />$dark-overlay: rgba(37, 36, 35, 0.75);<br />$dark-surfaceBackground: #2d2c2c;<br />$dark-primaryText: #ffffff;<br />$dark-panelBorder: #4c4b4b;<br /><br />// contrast theme<br />$contrast-overlay: rgba(37, 36, 35, 0.75);<br />$contrast-surfaceBackground: #000000;<br />$contrast-primaryText: #ffffff;<br />$contrast-panelBorder: #4c4b4b;<br /><br /></pre>
<br /><span class="code">Global.default.module.scss</span>
<pre
    class="brush: css;"><br />@import './colors.module';<br />[data-theme='default'] {<br />  :global {<br />    .ms-Fabric {<br />      color: $default-primaryText;<br />    }<br />    .ms-Button-icon {<br />      color: $default-primaryText;<br />    }<br /><br />    .ms-Overlay {<br />      background-color: $default-overlay;<br />    }<br /><br />    .ms-Panel-main {<br />      background-color: $default-surfaceBackground;<br />      border-left-color: $default-panelBorder;<br />      border-right-color: $default-panelBorder;<br />      .ms-Panel-headerText {<br />        color: $default-primaryText;<br />      }<br />    }<br />  }<br />}<br /></pre>
<br /><span class="code">Global.dark.module.scss</span>
<pre
    class="brush: css;"><br />@import './colors.module';<br />[data-theme='dark'] {<br />  :global {<br />    .ms-Fabric {<br />      color: $dark-primaryText;<br />    }<br />    .ms-Button-icon {<br />      color: $dark-primaryText;<br />    }<br /><br />    .ms-Overlay {<br />      background-color: $dark-overlay;<br />    }<br /><br />    .ms-Panel-main {<br />      background-color: $dark-surfaceBackground;<br />      border-left-color: $dark-panelBorder;<br />      border-right-color: $dark-panelBorder;<br />      .ms-Panel-headerText {<br />        color: $dark-primaryText;<br />      }<br />    }<br />  }<br />}<br /></pre>
<br /><span class="code">Global.contrast.module.scss</span>
<pre
    class="brush: css;"><br />@import './colors.module';<br />[data-theme='contrast'] {<br />  :global {<br />    .ms-Fabric {<br />      color: $contrast-primaryText;<br />    }<br />    .ms-Button-icon {<br />      color: $contrast-primaryText;<br />    }<br /><br />    .ms-Overlay {<br />      background-color: $contrast-overlay;<br />    }<br /><br />    .ms-Panel-main {<br />      background-color: $contrast-surfaceBackground;<br />      border-left-color: $contrast-panelBorder;<br />      border-right-color: $contrast-panelBorder;<br />      .ms-Panel-headerText {<br />        color: $contrast-primaryText;<br />      }<br />    }<br />  }<br />}<br /></pre>
And in the root component:<br />
<pre
    class="brush: css;"><br />@import '../../../common/Global.dark.module.scss';<br />@import '../../../common/Global.default.module.scss';<br />@import '../../../common/Global.contrast.module.scss';<br /></pre>
Now the Panel has correct colors as well:<br /><a
    href="https://2.bp.blogspot.com/-jwKGTDQfxbo/Xn_FkPW_ubI/AAAAAAAABl8/yPYm3m64it4IL1W2boduZ__FdwwfSlJ8wCLcBGAsYHQ/s1600/Screen%2BShot%2B2020-03-28%2Bat%2B2.45.29%2BPM.png"
    imageanchor="1"><img border="0"
        src="https://2.bp.blogspot.com/-jwKGTDQfxbo/Xn_FkPW_ubI/AAAAAAAABl8/yPYm3m64it4IL1W2boduZ__FdwwfSlJ8wCLcBGAsYHQ/s1600/Screen%2BShot%2B2020-03-28%2Bat%2B2.45.29%2BPM.png"
        data-original-width="1422" data-original-height="702" width="700" /></a><br />
<h2>6. Don't Forget About Web Part Property Pane!</h2>One component that still looks ugly is Web Part Property
Pane<br /><a
    href="https://3.bp.blogspot.com/-kXGTD6MziXo/Xn_F9OcQ_zI/AAAAAAAABmE/3ORncDZfhtkWOixZDkrB7CzATgnnn-AMQCLcBGAsYHQ/s1600/Screen%2BShot%2B2020-03-28%2Bat%2B2.47.02%2BPM.png"
    imageanchor="1"><img border="0"
        src="https://3.bp.blogspot.com/-kXGTD6MziXo/Xn_F9OcQ_zI/AAAAAAAABmE/3ORncDZfhtkWOixZDkrB7CzATgnnn-AMQCLcBGAsYHQ/s1600/Screen%2BShot%2B2020-03-28%2Bat%2B2.47.02%2BPM.png"
        data-original-width="1600" data-original-height="674" width="700" /></a><br />And, unfortunately, it doesn't
have any global classes we can override. Only <span class="code">.spPropertyPaneContainer</span>.<br />But of course we
can use other CSS selectors.<br />Again, let's define all the colors first in our <span
    class="code">_colors.module.scss</span>:
<pre class"brush:
    typescript;"><br />// SharePoint<br />$white: "[theme:white, default: #fff]"; // property pane background<br />$inputBackground: "[theme:inputBackground, default:#fff]"; //input background<br />$inputBorder: "[theme:inputBorder, default:#a6a6a6]"; // input border<br />$inputBorderHovered: "[theme:inputBorderHovered, default:#333333]"; // input border hovered<br /><br />// default theme<br />$default-white: #f3f2f1;<br />$default-inputBackground: #fff;<br />$default-inputBorder: #b5b4b2;<br />$default-inputBorderHovered: #252423;<br /><br />// dark-theme<br />$dark-white: #2d2c2c;<br />$dark-inputBackground: #000;<br />$dark-inputBorder: #c8c8c8;<br />$dark-inputBorderHovered: #ffffff;<br /><br />//contrast theme<br />$contrast-white: #000000;<br />$contrast-inputBackground: #000;<br />$contrast-inputBorder: #c8c8c8;<br />$contrast-inputBorderHovered: #ffffff;<br /></pre>
And now let's add overrides to <span class="code">Global.default.module.scss, Global.dark.module.scss,
    Global.contrast.module.scss</span> (<b>Note: the CSS below should be added inside <span class="code">[data-theme] {
        :global {</span></b>): <span class="code">Global.default.module.scss</span>
<pre
    class="brush: css;"><br />// Property Pane<br />.spPropertyPaneContainer {<br />  background-color: $default-white;<br />  [class^="propertyPane_"] {<br />    background-color: $default-white;<br />    border-left-color: $default-panelBorder;<br />    [class^="propertyPanePageTitle_"],<br />    [class^="propertyPanePageDescription_"],<br />    [class^="propertyPaneGroupHeaderNoAccordion_"] {<br />      color: $default-primaryText;<br />    }<br />    .ms-Button--icon {<br />      &:hover {<br />        background-color: transparent;<br />      }<br />    }<br />  }<br />}<br /><br />// Text Field<br />.ms-Label {<br />  color: $default-primaryText;<br />}<br />.ms-TextField {<br />  .ms-TextField-fieldGroup {<br />    background-color: $default-inputBackground;<br />    color: $default-primaryText;<br />    border-color: $default-inputBorder;<br />    .ms-TextField-field {<br />      color: $default-primaryText;<br />    }<br />    &:hover {<br />      border-color: $default-inputBorderHovered;<br />    }<br />  }<br />}<br /></pre>
<br /><span class="code">Global.dark.module.scss</span>
<pre
    class="brush: css;"><br />// Property Pane<br />.spPropertyPaneContainer {<br />  background-color: $dark-white;<br />  [class^="propertyPane_"] {<br />    background-color: $dark-white;<br />    border-left-color: $dark-panelBorder;<br />    [class^="propertyPanePageTitle_"],<br />    [class^="propertyPanePageDescription_"],<br />    [class^="propertyPaneGroupHeaderNoAccordion_"] {<br />      color: $dark-primaryText;<br />    }<br />    .ms-Button--icon {<br />      &:hover {<br />        background-color: transparent;<br />      }<br />    }<br />  }<br />}<br /><br />// Text Field<br />.ms-Label {<br />  color: $dark-primaryText;<br />}<br />.ms-TextField {<br />  .ms-TextField-fieldGroup {<br />    background-color: $dark-inputBackground;<br />    color: $dark-primaryText;<br />    border-color: $dark-inputBorder;<br />    .ms-TextField-field {<br />      color: $dark-primaryText;<br />    }<br />    &:hover {<br />      border-color: $dark-inputBorderHovered;<br />    }<br />  }<br />}<br /></pre>
<br /><span class="code">Global.contrast.module.scss</span>
<pre
    class="brush: css;"><br />// Property Pane<br />.spPropertyPaneContainer {<br />  background-color: $contrast-white;<br />  [class^="propertyPane_"] {<br />    background-color: $contrast-white;<br />    border-left-color: $contrast-panelBorder;<br />    [class^="propertyPanePageTitle_"],<br />    [class^="propertyPanePageDescription_"],<br />    [class^="propertyPaneGroupHeaderNoAccordion_"] {<br />      color: $contrast-primaryText;<br />    }<br />    .ms-Button--icon {<br />      &:hover {<br />        background-color: transparent;<br />      }<br />    }<br />  }<br />}<br /><br />// Text Field<br />.ms-Label {<br />  color: $contrast-primaryText;<br />}<br />.ms-TextField {<br />  .ms-TextField-fieldGroup {<br />    background-color: $contrast-inputBackground;<br />    color: $contrast-primaryText;<br />    border-color: $contrast-inputBorder;<br />    .ms-TextField-field {<br />      color: $contrast-primaryText;<br />    }<br />    &:hover {<br />      border-color: $contrast-inputBorderHovered;<br />    }<br />  }<br />}<br /></pre>
<br />Yay! Now all the parts of our web part look amazing:<br /><a
    href="https://3.bp.blogspot.com/-QGO4jWFoGF8/XoD3dtKfnlI/AAAAAAAABm8/5uk94NH1JxIRo-p5wIvi7Q-Px7wLCN2OACLcBGAsYHQ/s1600/Screen%2BShot%2B2020-03-29%2Bat%2B12.30.30%2BPM.png"
    imageanchor="1"><img border="0"
        src="https://3.bp.blogspot.com/-QGO4jWFoGF8/XoD3dtKfnlI/AAAAAAAABm8/5uk94NH1JxIRo-p5wIvi7Q-Px7wLCN2OACLcBGAsYHQ/s1600/Screen%2BShot%2B2020-03-29%2Bat%2B12.30.30%2BPM.png"
        data-original-width="1600" data-original-height="670" width="700" /></a><br /><br />
<h2>Conclusion</h2>As you can see it takes time to support MS Teams team in your SharePoint Framework web part. Even if
you have a single component in there. Imagine how long it will take to add support for 50 components.<br />So, I would
recommend to add support in the moment when you develop each component. It will be much easier and not so painful.
<br />Hopefully, this post will also reduce the time you spend to support Teams themes in your SPFx solutions.<br />The
code sample for this post can be found <a href="https://github.com/AJIXuMuK/SPFx/tree/master/teams-themes"
    target="_blank">here</a>. <br /><br />That's all for today!<br />Have fun!