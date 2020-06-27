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
{% highlight sass %}
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
{% highlight typescript %}
protected async onInit(): Promise<void> {
    if (this.context.sdks.microsoftTeams) { 
        // checking that we're in Teams
        const context = this.context.sdks.microsoftTeams!.context;
        this._applyTheme(context.theme || 'default');
        this.context.sdks.microsoftTeams.teamsJs.registerOnThemeChangeHandler(this._applyTheme);
    }
}
private _applyTheme = (theme: string): void => {
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
    <div markdonw="1">
{% highlight sass %}
        .dark {
            .firstComponent {        
            }
        }
{% endhighlight %}
</div>
    </li>
    <li><b>You can use global class name instead of data attribute</b>. I just prefer data attribute.</li>
</ul>
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

<div markdown="1">
{% highlight sass %}
.firstComponent {
    background: "[theme:white, default:#fff]";
}
{% endhighlight %}
</div>

We can use variable:
<div markdown="1">
{% highlight sass %}
$background: "[theme:white, default:#fff]";
.firstComponent {
    background: $background;
}
{% endhighlight %}
</div>
Moreover, these variables can be define in a separate scss file and shared between different components.<br />I would
also recommend to provide pretty specific names for the variables. For example, if you want to use some color as <span
    class="code">FirstComponent</span> background, name the variable <span
    class="code">$firstComponent-background</span>.<br />Let's create variables for all our custom color:
<div markdown="1">
{% highlight sass %}
//SharePoint
$firstComponent-background: "[theme:white, default:#fff]";
$firstComponent-color: "[theme:primaryText, default:#333]";
$firstComponentButton-background: "[theme:themePrimary, default:#0078d4]";
$firstComponentButton-color: "[theme:white, default:#fff]";

// default theme
$default-firstComponent-background: #f3f2f1;
$default-firstComponent-color: #252423;
$default-firstComponentButton-background: #6264a7;
$default-firstComponentButton-color: #f3f2f1;

// dark theme
$dark-firstComponent-background: #2d2c2c;
$dark-firstComponent-color: #ffffff;
$dark-firstComponentButton-background: #6264a7;
$dark-firstComponentButton-color: #2d2c2c;

// contrast theme
$contrast-firstComponent-background: #000000;
$contrast-firstComponent-color: #ffffff;
$contrast-firstComponentButton-background: #6264a7;
$contrast-firstComponentButton-color: #000000;
{% endhighlight %}
</div>
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
<div markdown="1">
{% highlight sass %}
@import "../../common/colors.module";
.firstComponent {
    background: "[theme:white, default:#fff]";
    color: "[theme:primaryText, default:#333]";
    
    .button {
        background: "[theme:themePrimary, default:#0078d4]";
        color: "[theme:white, default:#fff]";
    }
}

[data-theme='default'] {
    .firstComponent {
        background: $default-firstComponent-background;
        color: $default-firstComponent-color;
        .button {
            background: $default-firstComponentButton-background;
            color: $default-firstComponentButton-color;
        }
    }
}

[data-theme='dark'] {
    .firstComponent {
        background: $dark-firstComponent-background;
        color: $dark-firstComponent-color;
        
        .button {
            background: $dark-firstComponentButton-background;
            color: $dark-firstComponentButton-color;
        }
    }
}

[data-theme='contrast'] {
    .firstComponent {
        background: $contrast-firstComponent-background;
        color: $contrast-firstComponent-color;
        
        .button {
            background: $contrast-firstComponentButton-background;
            color: $contrast-firstComponentButton-color;
        }
    }
}
{% endhighlight %}
</div>
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
<div markdown="1">
{% highlight sass %}
//SharePoint
$overlay: "[theme:whiteTranslucent40, default:rgba(255, 255,255, 0.4)]";
$surfaceBackground: "[theme:white, default:#fff]";
$primaryText: "[theme:primaryText, default:#333]";
$panelBorder: "[theme: neutralLight, default: #eaeaea]";

// default theme
$default-overlay: rgba(255, 255, 255, 0.4);
$default-surfaceBackground: #f3f2f1;
$default-primaryText: #252423;
$default-panelBorder: #dedddc;

// dark theme
$dark-overlay: rgba(37, 36, 35, 0.75);
$dark-surfaceBackground: #2d2c2c;
$dark-primaryText: #ffffff;
$dark-panelBorder: #4c4b4b;

// contrast theme
$contrast-overlay: rgba(37, 36, 35, 0.75);
$contrast-surfaceBackground: #000000;
$contrast-primaryText: #ffffff;$contrast-panelBorder: #4c4b4b;
{% endhighlight %}
</div>
<br /><span class="code">Global.default.module.scss</span>
<div markdown="1">
{% highlight sass %}
@import './colors.module';
[data-theme='default'] {
    :global {
        .ms-Fabric {
            color: $default-primaryText;
        }
        .ms-Button-icon {
            color: $default-primaryText;
        }
        
        .ms-Overlay {
            background-color: $default-overlay;
        }
        
        .ms-Panel-main {
            background-color: $default-surfaceBackground;
            border-left-color: $default-panelBorder;
            border-right-color: $default-panelBorder;
            .ms-Panel-headerText {
                color: $default-primaryText;
            }
        }
    }
}
{% endhighlight %}
</div>
<br /><span class="code">Global.dark.module.scss</span>
<div markdown="1">
{% highlight sass %}
@import './colors.module';
[data-theme='dark'] {
    :global {
        .ms-Fabric {
            color: $dark-primaryText;
        }
        .ms-Button-icon {
            color: $dark-primaryText;
        }
        .ms-Overlay {
            background-color: $dark-overlay;
        }
        
        .ms-Panel-main {
            background-color: $dark-surfaceBackground;
            border-left-color: $dark-panelBorder;
            border-right-color: $dark-panelBorder;
            .ms-Panel-headerText {
                color: $dark-primaryText;
            }
        }
    }
}
{% endhighlight %}
</div>
<br />
<span class="code">Global.contrast.module.scss</span>
<div markdown="1">
{% highlight sass %}
@import './colors.module';
[data-theme='contrast'] {
    :global {
        .ms-Fabric {
            color: $contrast-primaryText;
        }
        .ms-Button-icon {
            color: $contrast-primaryText;
        }
        .ms-Overlay {
            background-color: $contrast-overlay;
        }
        
        .ms-Panel-main {
            background-color: $contrast-surfaceBackground;
            border-left-color: $contrast-panelBorder;
            border-right-color: $contrast-panelBorder;
            .ms-Panel-headerText {
                color: $contrast-primaryText;
            }
        }
    }
}
{% endhighlight %}
</div>
And in the root component:<br />
<div markdown="1">
{% highlight sass %}
@import '../../../common/Global.dark.module.scss';
@import '../../../common/Global.default.module.scss';
@import '../../../common/Global.contrast.module.scss';
{% endhighlight %}
</div>
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
<div markdown="1">
{% highlight sass %}
// SharePoint
$white: "[theme:white, default: #fff]"; // property pane background
$inputBackground: "[theme:inputBackground, default:#fff]"; //input background
$inputBorder: "[theme:inputBorder, default:#a6a6a6]"; // input border
$inputBorderHovered: "[theme:inputBorderHovered, default:#333333]"; // input border hovered

// default theme
$default-white: #f3f2f1;
$default-inputBackground: #fff;
$default-inputBorder: #b5b4b2;
$default-inputBorderHovered: #252423;

// dark-theme
$dark-white: #2d2c2c;
$dark-inputBackground: #000;
$dark-inputBorder: #c8c8c8;
$dark-inputBorderHovered: #ffffff;

//contrast theme
$contrast-white: #000000;
$contrast-inputBackground: #000;
$contrast-inputBorder: #c8c8c8;
$contrast-inputBorderHovered: #ffffff;
{% endhighlight %}
</div>
And now let's add overrides to <span class="code">Global.default.module.scss, Global.dark.module.scss,
    Global.contrast.module.scss</span> (<b>Note: the CSS below should be added inside <span class="code">[data-theme] {
        :global {</span></b>): <span class="code">Global.default.module.scss</span>
<div markdown="1">
{% highlight sass %}
// Property Pane
.spPropertyPaneContainer {
    background-color: $default-white;
    [class^="propertyPane_"] {
        background-color: $default-white;
        border-left-color: $default-panelBorder;
        [class^="propertyPanePageTitle_"],
        [class^="propertyPanePageDescription_"],
        [class^="propertyPaneGroupHeaderNoAccordion_"] {
            color: $default-primaryText;
        }
        .ms-Button--icon {
            &:hover {
                background-color: transparent;
            }
        }
    }
}

// Text Field
.ms-Label {
    color: $default-primaryText;
}
.ms-TextField {
    .ms-TextField-fieldGroup {
        background-color: $default-inputBackground;
        color: $default-primaryText;
        border-color: $default-inputBorder;
        .ms-TextField-field {
            color: $default-primaryText;
        }
        &:hover {
            border-color: $default-inputBorderHovered;
        }
    }
}
{% endhighlight %}
</div>
<br /><span class="code">Global.dark.module.scss</span>
<div markdown="1">
{% highlight sass %}
// Property Pane
.spPropertyPaneContainer {
    background-color: $dark-white;
    [class^="propertyPane_"] {
        background-color: $dark-white;
        border-left-color: $dark-panelBorder;
        [class^="propertyPanePageTitle_"],
        [class^="propertyPanePageDescription_"],
        [class^="propertyPaneGroupHeaderNoAccordion_"] {
            color: $dark-primaryText;
        }
        .ms-Button--icon {
            &:hover {
                background-color: transparent;
            }
        }
    }
}

// Text Field
.ms-Label {
    color: $dark-primaryText;
}
.ms-TextField {
    .ms-TextField-fieldGroup {
        background-color: $dark-inputBackground;
        color: $dark-primaryText;
        border-color: $dark-inputBorder;
        .ms-TextField-field {
            color: $dark-primaryText;
        }
        &:hover {
            border-color: $dark-inputBorderHovered;
        }
    }
}
{% endhighlight %}
</div>
<br /><span class="code">Global.contrast.module.scss</span>
<div markdown="1">
{% highlight sass %}
// Property Pane
.spPropertyPaneContainer {
    background-color: $contrast-white;
    [class^="propertyPane_"] {
        background-color: $contrast-white;
        border-left-color: $contrast-panelBorder;
        [class^="propertyPanePageTitle_"],
        [class^="propertyPanePageDescription_"],
        [class^="propertyPaneGroupHeaderNoAccordion_"] {
            color: $contrast-primaryText;
        }
        .ms-Button--icon {
            &:hover {
                background-color: transparent;
            }
        }
    }
}

// Text Field
.ms-Label {
    color: $contrast-primaryText;
}
.ms-TextField {
    .ms-TextField-fieldGroup {
        background-color: $contrast-inputBackground;
        color: $contrast-primaryText;
        border-color: $contrast-inputBorder;
        .ms-TextField-field {
            color: $contrast-primaryText;
        }
        &:hover {
            border-color: $contrast-inputBorderHovered;
        }
    }
}
{% endhighlight %}
</div>
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