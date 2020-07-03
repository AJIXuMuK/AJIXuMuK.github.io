---
layout: post
title: Office UI Fabric React Grouped DetailsList - Display Tree-like Hierarchy
date: '2020-02-12T14:42:00.000-08:00'
author: Alex Terentiev
tags:
- Office UI Fabric
- DetalisList
- SharePoint Framework
- Office 365
- Grid
- SharePoint
- Office UI Fabric React
- SPFx
- SPFx Web Parts
- OUIFR
- Tree
- Hierarchy
modified_time: '2020-02-12T14:49:06.622-08:00'
blogger_id: tag:blogger.com,1999:blog-3066084330774405472.post-5476940037224468383
blogger_orig_url: http://blog.aterentiev.com/2020/02/office-ui-fabric-react-grouped.html
---

This blog post describes how to implement tree-like hierarchy using Office UI Fabric React (OUIFR) Grouped DetailsList component.<br />We'll be working with a hierarchy similar to the one displayed on the image below:<br /><img border="0" src="{{site.baseurl}}/assets/images/posts/2020/hierarchy.png" /><br /><h2>TL;DR</h2>Office UI Fabric React (OUIFR) DetailsList ignores (doesn't render) items for the group if it (the group) has subgroups. As a result for a tree-like hierarchy - some of the items could be lost.<br /><a href="#solution">Jump to solution</a>.  <br /><a name='more'></a><h2>The Problem</h2>We need to display a hierarchical grid, or grouped grid where each node (group) can have child nodes as well as leaf items.<br />For the hierarchy from the picture above we want to see something like this:<br /><img border="0" src="{{site.baseurl}}/assets/images/posts/2020/design.png" /><br />So, we have a root level node <b>code</b> that contains leaf item <b>index.ts</b> and two subgroups with their own items: <b>components</b> and <b>webparts</b>.<br />As we're developing SharePoint Framework solution (of course - this blog is about O365 and SharePoint =)) it would be great to use OUIFR <span class="code">DetailsList</span> component to display grouped grid with a header. <br /><h2>Initial Implementation</h2>Initial implementation for our component will be pretty simple.<br />We need to define our leaf item interface: <br />
<div markdown="1">
{% highlight typescript %}
export interface IItem {
  key: string;
  title: string;
}

{% endhighlight %}
</div>
And our node interface: <br />
<div markdown="1">
{% highlight typescript %}
export interface INode extends IItem {
  items?: IItem[];
  children?: INode[];
}

{% endhighlight %}
</div>
Using these two interfaces we can define our hierarchy like that: <br />
<div markdown="1">
{% highlight typescript %}
private readonly _nodes: INode[] = [{
    key: 'code',
    title: 'code',
    items: [{
      key: 'index',
      title: 'index.ts'
    }],
    children: [{
      key: 'components',
      title: 'components',
      items: [{
        key: 'component',
        title: 'Component.tsx'
      }]
    }, {
      key: 'webparts',
      title: 'webparts',
      items: [{
        key: 'scss',
        title: 'WebPart.module.scss'
      }, {
        key: 'ts',
        title: 'WebPart.ts'
      }]
  }]
}];

{% endhighlight %}
</div>
 Now, we need to process our hierarchy to construct flat array of <span class="code">IItem</span> items and array of <span class="code"><a href="https://developer.microsoft.com/en-us/fabric#/controls/web/groupedlist#IGroup" target="_blank">IGroup</a></span> groups. This arrays will be used by <span class="code">DetailsList</span> component to display <a href="https://developer.microsoft.com/en-us/fabric#/controls/web/detailslist/grouped" target="_blank">grouped list</a>.<br />Let's assume that the hierarchy is passed in the <span class="code">props</span> of our component: 
<div markdown="1">
{% highlight typescript %}

export interface IOuifrGroupedDetailsListProps {
  nodes: INode[];
}

{% endhighlight %}
</div>
Then, we can used the methods below to process the hierarchy and save results to the state: 
<div markdown="1">
{% highlight typescript %}

/**
 * Gets flat items array and groups array based on the hierarchy from the props
 */
private _getItemsAndGroups = (props: IOuifrGroupedDetailsListProps): void => {
  const nodes = props.nodes;
  const items: IItem[] = [];
  const groups: IGroup[] = [];

  // processing all the nodes recursively
  this._processNodes(nodes, groups, items, 0);

  // setting the state
  this.setState({
    groups: groups,
    items: items
  });
}

/**
 * Recursively process hierarchy's nodes to build groups and add items to the flat array
 */
private _processNodes = (nodeItems: INode[] | undefined, groups: IGroup[], items: IItem[], level: number): void => {
  // end of recursion
  if (!nodeItems || !nodeItems.length) {
    return;
  }

  // processing current level of the tree
  nodeItems.forEach(nodeItem => {
    const newGroup: IGroup = {
      key: nodeItem.key,
      name: nodeItem.title,
      startIndex: items.length,
      count: 0,
      children: [],
      level: level, // level is incremented on each call of the recursion
      data: nodeItem // storing initial INode instance in the group's data
    };

    groups.push(newGroup);
    if (nodeItem.items && nodeItem.items.length) {

      // adding items to the flat array
      items.push(...nodeItem.items);
    }

    // processing child nodes
    this._processNodes(nodeItem.children, newGroup.children!, items, level + 1);

    // current group count is a sum of group's leaf items and leaf items in all child nodes
    newGroup.count = items.length - newGroup.startIndex;
  });
}

{% endhighlight %}
</div>
 And now we can render the <span class="code">DetailsList</span> using values from the state: 
<div markdown="1">
{% highlight typescript %}

public render(): React.ReactElement<IOuifrGroupedDetailsListProps> {
  const {
    items,
    groups
  } = this.state;

  return (
    <div className={styles.ouifrGroupedDetailsList}>
      <DetailsList
          columns={this._columns}
          items={items || []}
          groups={groups}
        />
    </div>
  );
}

{% endhighlight %}
</div>
If we look at the rendered result, we'll see such a list:<br /><img border="0" src="{{site.baseurl}}/assets/images/posts/2020/details-list.png" /><br />The problem here is we're missing <b>index.ts</b> leaf item in the <b>code</b> group. So, it's kinda data loss situation from user perspective.<br />The reason for that, as mentioned above, is <span class="code">DetailsList</span> component ignores group's items if there are subgroups. <a name="solution"></a><h2>The Solution</h2>The solution here consists of two parts.<br />First, we'll add "fake" subgroup and place all the missing leaf items in it. For our sample we'll add <b>fake</b> subgroup to <b>code</b> group and move <b>index.ts</b> into it: 
<div markdown="1">
{% highlight typescript %}

private _processNodes = (nodeItems: INode[] | undefined, groups: IGroup[], items: IItem[], level: number): void => {
  // ...

  // processing current level of the tree
  nodeItems.forEach(nodeItem => {
    // ...
    groups.push(newGroup);
    if (nodeItem.items && nodeItem.items.length) {
      // adding fake group with no data
      if (nodeItem.children && nodeItem.children.length) {
        newGroup.children!.push({
          key: `${nodeItem.key}-fake`,
          name: '',
          startIndex: items.length,
          count: nodeItem.items.length,
          level: level
        });
      }

      // adding items to the flat array
      items.push(...nodeItem.items);
    }

    // ...
  });
}

{% endhighlight %}
</div>
The fake group doesn't have <span class="code">data</span> as there is no actual <span class="code">INode</span> item related to it.<br />Now our component looks like that:<br /><img border="0" src="{{site.baseurl}}/assets/images/posts/2020/fake-group.png" /><br />The <b>index.ts</b> item is there, but, as expected, it is displayed as a child of <b>fake</b> group.<br /><br />The second step of the solution is to override rendering of group's header and hide the header for any fake group.<br />As we know, fake groups don't have <span class="code">data</span> assigned, so we can use it to determine if the group is fake.<br />And we can override group's header rendering using <span class="code">groupProps</span> property of the <span class="code">DetailsList</span>: 
<div markdown="1">
{% highlight typescript %}

public render(): React.ReactElement<IOuifrGroupedDetailsListProps> {
  // ...
  return (
    <div className={ styles.ouifrGroupedDetailsList }>
      <DetailsList
          /* ... */
          groupProps={{ "{{" }}
            onRenderHeader: this._onRenderGroupHeader,
            isAllGroupsCollapsed: groups ? groups.filter(gr => !gr.isCollapsed).length === 0 : true,
            collapseAllVisibility: CollapseAllVisibility.visible
          }}
      />
    </div>
  );
}

private _onRenderGroupHeader = (props: IDetailsGroupDividerProps, _defaultRender?: IRenderFunction<IDetailsGroupDividerProps>): JSX.Element => {
  // for fake groups - return empty element
  if (!props.group!.data) {
    return <></>;
  }

  // default rendering for "real" groups
  return _defaultRender(props);
}

{% endhighlight %}
</div>
After these changes we'll see the result we actually want. Moreover, all events, including collapsing/expanding will still work as expected!<br /><br />You can find full sample code for this post <a href="https://github.com/AJIXuMuK/SPFx/tree/master/ouifr-grouped-details-list" target="_blank">here</a>. <br /><br />That's all for today!<br />Have fun!