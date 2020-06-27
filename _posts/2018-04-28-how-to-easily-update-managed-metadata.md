---
layout: post
title: 'How To: Easily Update Managed Metadata Field Value (... and others) via REST
  API/PnPJS'
date: '2018-04-28T11:08:00.000-07:00'
author: Alex Terentiev
tags:
- JavaScript
- SharePoint Online
- REST
- TypeScript
- REST API
- PnPJS
- PnP
- SharePoint
modified_time: '2018-04-28T11:09:00.287-07:00'
blogger_id: tag:blogger.com,1999:blog-3066084330774405472.post-6143235659454687317
blogger_orig_url: http://blog.aterentiev.com/2018/04/how-to-easily-update-managed-metadata.html
---

It's a short post that describes how to easily update Managed Metadata Field value using SharePoint REST API or PnPJS.<br /><br /><a name='more'></a><br />Although the Title of the post is about REST API and PnPJS I'll describe the process using PnPJS only as it is easier and absolutely identical to REST API.<br />Usually, to update list item we use <span class="code">update</span> for the list item like that: <br />
<div markdown="1">
{% highlight typescript %}
pnp.sp.web.lists.getByTitle('List').items.getById(1).update({
  Title: 'New Title',
  CustomField: 'Field Value'
});

{% endhighlight %}
</div>
The documentation for the <span class="code">update</span> method can be found <a href="https://github.com/SharePoint/PnP-JS-Core/wiki/Working-With:-Items#update" target="_blank">here</a><br />But this method is not very convenient when we want to update Managed Metadata Field value. And especially if the field can contain multiple terms selected.<br />Even for a single term the call will look like that: <br />
<div markdown="1">
{% highlight typescript %}
pnp.sp.web.lists.getByTitle('List').items.getById(1).update({
  MetaData: {
    __metadata: { "type": "SP.Taxonomy.TaxonomyFieldValue" },
    Label: "1",
    TermGuid: "6eccf028-399c-486d-9260-72c81f2d5344",
    WssId: -1
  }
});

{% endhighlight %}
</div>
where <span class="code">Label</span> is not a term label but its ID. If you don't have terms localization it will be "1". But anyway, it's not really cool to construct such objects, right?<br />That's why I would recommend to use another available method: <span class="code">validateUpdateListItem</span>.<br />In this method we need to provide array of objects with the next properties: <br />
<div markdown="1">
{% highlight typescript %}
{
  ErrorMessage: null,
  FieldName: your_field_name,
  FieldValue: field_value,
  HasException: false
}

{% endhighlight %}
</div>
And for the Managed Metadata Field the <span class="code">FieldValue</span> value can be constructed really easily: <span class="code">term_label + "|" + term_id + ";"</span><br />And this format works both for single value and multiple values<br />So, for example we have <span class="code">Categories</span> field and we want to set the value to contain terms "HR" and "Benefits". We can do that like this: <br />
<div markdown="1">
{% highlight typescript %}
pnp.sp.web.lists.getByTitle('List').items.getById(1).validateUpdateListItem([{
  ErrorMessage: null,
  FieldName: "Categories",
  FieldValue: "HR|6192598e-8273-4527-89f1-2cd9becb078e;Benefits|6c2d4c17-080a-4a41-b022-766f068fcf73",
  HasException: false
}]);

{% endhighlight %}
</div>
If you want to update multiple fields, just add them to the array: <br />
<div markdown="1">
{% highlight typescript %}
pnp.sp.web.lists.getByTitle('List').items.getById(1).validateUpdateListItem([{
  ErrorMessage: null,
  FieldName: "Categories",
  FieldValue: "HR|6192598e-8273-4527-89f1-2cd9becb078e;Benefits|6c2d4c17-080a-4a41-b022-766f068fcf73",
  HasException: false
}, {
  ErrorMessage: null,
  FieldName: "Title",
  FieldValue: "New Title",
  HasException: false
}]);

{% endhighlight %}
</div>
As a cocnlusion, for simple field types <span class="code">update</span> method is easier, but for such fields as Managed Metadata <span class="code">validateUpdateListItem</span> is better.<br /><br />That's it!<br />Have fun!