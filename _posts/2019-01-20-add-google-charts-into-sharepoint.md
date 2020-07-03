---
layout: post
title: Add Google Charts into SharePoint Framework Web Part
date: '2019-01-20T11:46:00.000-08:00'
author: Alex Terentiev
tags:
- SharePoint Online
- Client Side Web Part
- SPFx
- O365
- Google Charts
- SharePoint Framework
- Office 365
- SharePoint
modified_time: '2019-01-20T11:46:21.355-08:00'
featured_image_thumbnail: assets/images/posts/2019/google-charts.png
blogger_id: tag:blogger.com,1999:blog-3066084330774405472.post-2015303535837982937
blogger_orig_url: http://blog.aterentiev.com/2019/01/add-google-charts-into-sharepoint.html
---

This post is based on the <a href="https://github.com/SharePoint/sp-dev-docs/issues/3298" target="_blank">question</a> in sp-dev-docs repo.<br />The question is how can we render a Google Charts in SPFx web part.<br />Actually, it's pretty easy thing to do. <br /><a name='more'></a>As a starting point I'll be using the code provided in the question I mentioned above:<br />
<div markdown="1">
{% highlight javascript %}

<script type="text/javascript" src="https://www.gstatic.com/charts/loader.js"></script>
<script type="text/javascript"> 
google.charts.load("current", {packages:["corechart"]}); 
google.charts.setOnLoadCallback(drawChart); 
function drawChart() { 
  var data = google.visualization.arrayToDataTable([ ['Task', 'Hours per Day'], ['Work', 11], ['Eat', 2], ['Commute', 2], ['Watch TV', 2], ['Sleep', 7] ]);
  var options = {
    title: 'My Daily Activities',
    pieHole: 0.4,
  };

  var chart = new google.visualization.PieChart(document.getElementById('donutchart'));
  chart.draw(data, options);
}
</script>

{% endhighlight %}
</div>
<br />So, we need to<br /><ul><li>Reference loader script from external CDN</li><li>Use that loader to load additional scripts for charts</li><li>Render a chart after all dependent scripts are loaded</li></ul>Let's apply it to SharePoint Framework!<br />Google Charts loader is a non-AMD script. So, we need to proceed with steps from <a href="https://docs.microsoft.com/en-us/sharepoint/dev/spfx/web-parts/basics/add-an-external-library#load-a-non-amd-module" target="_blank">official documentation</a> on how to load a non-AMD module.<br />First, let's update <span class="code">config/config.json</span> to reference external script with global namespace <span class="code">google</span>: 
<div markdown="1">
{% highlight javascript %}

"externals": {
  "google": {
    "path": "https://www.gstatic.com/charts/loader.js",
    "globalName": "google"
  }
}

{% endhighlight %}
</div>
Next, let's add typings for the script that will contain basic declaration of the module and available properties/functions. For that let's add <span class="code">typings</span> folder into <span class="code">src</span> and <span class="code">google.d.ts</span> file.<br />For the simplicity I've just added 2 properties into declaration: <span class="code">charts</span> and <span class="code">visualization</span>. 
<div markdown="1">
{% highlight typescript %}

declare module "google" {
    interface IGoogle {
        charts: any;
        visualization: any;
    }

    var google: IGoogle;
    export = google;
}

{% endhighlight %}
</div>
Now we can reference the module in our web part code: 
<div markdown="1">
{% highlight typescript %}

import * as google from 'google';

{% endhighlight %}
</div>
This import will also search for the external script declaration in our <span class="code">config.json</span> file and load <span class="code">loader.js</span> script from external CDN.<br />Next, let's modify web part's <span class="code">render</span> method to remove all unnecessary HTML and leave there a single <span class="code">div</span> with pre-defined id that will be used later to render a chart: 
<div markdown="1">
{% highlight typescript %}

public render(): void {
  this.domElement.innerHTML = `
    <div class="${ styles.helloWorld}" id="pie-chart">
    </div<`;
}

{% endhighlight %}
</div>
Next step is to implement chart rendering function that will be called after Google Charts scripts are loaded: 
<div markdown="1">
{% highlight typescript %}

private _drawChart() {
  const data = google.visualization.arrayToDataTable([ 
    ['Task', 'Hours per Day'], 
    ['Work', 11], 
    ['Eat', 2], 
    ['Commute', 2], 
    ['Watch TV', 2], 
    ['Sleep', 7] ]);
  const options = {
    title: 'My Daily Activities',
    pieHole: 0.4,
  };

  const chart = new google.visualization.PieChart(document.getElementById('pie-chart'));
  chart.draw(data, options);
}

{% endhighlight %}
</div>
Now we're ready to call Google Charts loader API to load needed dependencies and set callback function. I did that in <span class="code">onInit</span> method of the web part as it should be done only once and logically it's the right place to load additional resources for the web part: 
<div markdown="1">
{% highlight typescript %}

protected onInit(): Promise<void> {
  google.charts.load("current", { packages: ["corechart"] });
  google.charts.setOnLoadCallback(this._drawChart.bind(this));

  return super.onInit();
}

{% endhighlight %}
</div>
Now we can run our solution with <span class="code">gulp serve</span> and see Google Charts in action:<br /><img border="0" src="{{site.baseurl}}/assets/images/posts/2019/google-charts.png" /><br />And that's it! Google Charts are added to the SharePoint Framework web part!<br />You can find the code for this sample <a href="https://github.com/AJIXuMuK/issues/tree/master/google-charts" target="_blank">here</a>.<br />I want to mention that this sample is not final solution to be reused without modifications as it doesn't contain some additional must-haves like checking in <span class="code">_drawChart</span> if <span class="code">pie-chart</span> div has been added to the DOM and so on. So, think of it as a starting point for your implementation.<br /><br />That's it for today!<br />Have fun!