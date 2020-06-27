---
layout: post
title: Fixing executeQueryAsync "The file _list_or_something_else_ has been modified
  by _user_ on _date_" exception
date: '2015-02-11T16:58:00.002-08:00'
author: Alex Terentiev
tags:
- JavaScript
- SharePoint Online
- SharePoint 2013
- executeQueryAsync
- SharePoint 2010
- Client Side Object Model
- CSOM
- Queue
- Batch queries
- SharePoint
modified_time: '2015-02-11T17:01:57.926-08:00'
blogger_id: tag:blogger.com,1999:blog-3066084330774405472.post-1201720221058654455
blogger_orig_url: http://blog.aterentiev.com/2015/02/fixing-executequeryasync-file-list-or.html
---

The reason for the exception is the update of some entity in two separate queries that are executed at the same time.<br />For example, you're changing list item title and description. But you're not grouping these operations but executing them separately. In such situation you'll get the described error.<br /><br /><a name='more'></a>Best practices teach us to make all changes and then call executeQueryAsync. In most cases it is really the best and the most correct solution.<br />But what if you have such kind of API that there are functions that are enough independent and created to change some properties of the same objects. For example, some functions to change different properties from Property Bag. These functions can be called separately or inside some high-level function or event handler.<br />To my mind such functions have to call executeQueryAsync inside their body to be really completed and finished. But that returns us to the exception.<br />To secure yourself you can create a small helper object that allows to implement such kind of functions and to call them anywhere and anyhow.<br /><br />There are several things you have to worry about when thinking about execution of queries:<br /><br /><ol><li>You need some way to execute queries as batches</li><li>You have to store callbacks (success and fail) to call them after batch-query is executed</li><li>You need some queue if your latest batch is pending or someone has decided not to use batches.</li></ol><div><i><b>1. Execute queries as batches</b></i></div><div>First of all we need some methods to start a batch and to end it. For that we can implement standard approach: some counter that will be increased and decreased. And the query will be executed when the counter equals zero:</div><div>
<div markdown="1">
{% highlight javascript %}
beginBatch: function () {
  if (!this._updateCounter)
    // counter initialization
    this._updateCounter = 1;
  else
    // counter increment
    this._updateCounter++;
},
endBatch: function () {
  // decrement of the counter
  if (this._updateCounter) {
    this._updateCounter--;

    if (!this._updateCounter) {
      // query execution
      var spContext = SP.ClientContext.get_current();
      spContext.executeQueryAsync();
    }
  }
}

{% endhighlight %}
</div>
Now we have some function to be called from the code when there is need to execute query. Otherwise if we still use executeQueryAsync from ClientContext object there will no be any effect of using our beginBatch and endBatch functions.<br />
<div markdown="1">
{% highlight javascript %}
executeQueryAsync: function () {
  if (!this._updateCounter) {
      var spContext = SP.ClientContext.get_current();
      spContext.executeQueryAsync(successCallback, failCallback);
  }
}

{% endhighlight %}
</div>
Now there is not so much code here: we're just checking if the batch is started and executing query if it is not started. But we'll add some code here later.<br />So now if we want to execute some single operation we can just call helper.executeQueryAsync and if there is need to make a batch we can use beginBatch\endBatch.<br />Also there is a duplicate code in endBatch and executeQueryAsync. So let's make it a separate function:<br />
<div markdown="1">
{% highlight javascript %}
_executeQuery: function() {
  // query execution
  var spContext = SP.ClientContext.get_current();
  spContext.executeQueryAsync();
},
beginBatch: function () {
  if (!this._updateCounter)
    // counter initialization
    this._updateCounter = 1;
  else
    // counter increment
    this._updateCounter++;
},
endBatch: function () {
  // decrement of the counter
  if (this._updateCounter) {
    this._updateCounter--;

    if (!this._updateCounter) {
      this._executeQuery();
    }
  }
},
executeQueryAsync: function () {
  if (!this._updateCounter) {
      this._executeQuery();
  }
}

{% endhighlight %}
</div>
<br /><b><i>2. Store and execute success and fail callbacks</i></b><br />ClientContext.executeQueryCallback provides a possibility to handle success or fail query execution. And of course we must provide the same possibility for our batches. For that purposes we need to add arrays to store the callbacks and implement logic to call them after executing the query. Let's modify our last code and add some new:<br />
<div markdown="1">
{% highlight javascript %}
_executeQuery: function (successCalback, failCallback) {
  var spContext = SP.ClientContext.get_current();

  spContext.executeQueryAsync(
    jQuery.proxy(function (ctx, args) {
      successCalback.call(this, ctx, args);
    }, this),
    jQuery.proxy(function (ctx, args) {
      failCallback.call(this, ctx, args);
    }, this));
},

executeQueryAsync: function (successCallback, failCallback) {
  if (this._updateCounter) { // if we're in batch we just need to store the callbacks
    if (successCallback)
      this.addSuccessBatchQueryExecutionHandler(successCallback);
    if (failCallback)
      this.addFailBatchQueryExecutionHandler(failCallback);
  }
  else { // else we need to execute the query
    this._executeQuery(successCallback, failCallback);
  }
},

beginBatch: function () {
  if (!this._updateCounter)
    this._updateCounter = 1;
  else
    this._updateCounter++;
},

endBatch: function () {
  if (this._updateCounter) {
    this._updateCounter--;

    if (!this._updateCounter) {
      this._executeQuery(jQuery.proxy(function (context, args) {
        this.onBatchQueryExecuted(true, context, args);
      }, this),
      jQuery.proxy(function (context, args) {
        this.onBatchQueryExecuted(false, context, args);
      }, this));
    }
  }
},

//
// handler that is executed after the query is completed.
// It is used to call all callbacks
//
onBatchQueryExecuted: function (success, ctx, args) {
  var handlers;

  if (success) {
    handlers = this.successBatchHandlers;
  }
  else {
    handlers = this.failBatchHandlers;
  }

  if (handlers) {
    for (var i = 0, len = handlers.length; i < len; i++)
      handlers[i].call(this, ctx, args);
  }

  // we don't need callbacks anymore
  delete this.failBatchHandlers;
  delete this.successBatchHandlers;
},

addSuccessBatchQueryExecutionHandler: function (handler) {
  if (!handler)
    return;

  if (!this.successBatchHandlers)
    this.successBatchHandlers = [];

  this.successBatchHandlers.push(handler);
},

addFailBatchQueryExecutionHandler: function (handler) {
  if (!handler)
    return;

  if (!this.failBatchHandlers)
    this.failBatchHandlers = [];

  this.failBatchHandlers.push(handler);
}

{% endhighlight %}
</div>
<br /><b><i>3. Queries queue</i></b><br />Even after all the changes we've made it is still possible to initiate execution of queries more than one time at once. For example, the last query may execute too long. Or user (here our user is some developer that uses our code) doesn't know or has forgotten about batches.<br />In such situation the only way to make everything work correctly is to create a queue of queries. Actually it is not even a query. It's a flag that we need to execute something and some additional storage for callbacks. Or we even can just check if the additional storage is not empty and not to use any flag.<br />And we need some indicator to check if some query is pending and we need to store current information about callback to queue storage.<br />So here is a full code for our helper class:<br />
<div markdown="1">
{% highlight javascript %}
var Helper = {
  _executeQuery: function (successCalback, failCallback) {
    var spContext = SP.ClientContext.get_current();

    if (this._queryExecuting) // if the query is executing we don't need to do anything
      return;

    this._queryExecuting = true;
    spContext.executeQueryAsync(
      jQuery.proxy(function (ctx, args) {
        this._queryExecuting = false;
        this._executeQueriesFromQueue();
        successCalback.call(this, ctx, args);
      }, this),
      jQuery.proxy(function (ctx, args) {
        this._queryExecuting = false;
        this._executeQueriesFromQueue();
        failCallback.call(this, ctx, args);
      }, this));
  },

  executeQueryAsync: function (successCallback, failCallback) {
    if (this._updateCounter) { // if we're in batch we just need to store the callbacks
      if (successCallback)
        this.addSuccessBatchQueryExecutionHandler(successCallback);
      if (failCallback)
        this.addFailBatchQueryExecutionHandler(failCallback);
    }
    else if (this._queryExecuting) { // if some query is executing we need to store callbacks to queue storeage
      this.addItemToQueue(successCallback, failCallback);
    }
    else { // else we need to execute the query
      this._executeQuery(successCallback, failCallback);
    }
  },

  beginBatch: function () {
    if (!this._updateCounter)
      this._updateCounter = 1;
    else
      this._updateCounter++;
  },

  endBatch: function () {
    if (this._updateCounter) {
      this._updateCounter--;

      if (!this._updateCounter) {
        this._executeQuery(jQuery.proxy(function (context, args) {
          this.onBatchQueryExecuted(true, context, args);
        }, this),
        jQuery.proxy(function (context, args) {
          this.onBatchQueryExecuted(false, context, args);
        }, this));
      }
    }
  },

  onBatchQueryExecuted: function (success, ctx, args) {
    var handlers;

    if (success) {
      handlers = this.successBatchHandlers;
    }
    else {
      handlers = this.failBatchHandlers;
    }

    if (handlers) {
      for (var i = 0, len = handlers.length; i < len; i++)
        handlers[i].call(this, ctx, args);
    }

    // we don't need callbacks anymore
    delete this.failBatchHandlers;
    delete this.successBatchHandlers;
  },

  addSuccessBatchQueryExecutionHandler: function (handler) {
    if (!handler)
      return;

    if (!this.successBatchHandlers)
      this.successBatchHandlers = [];

      this.successBatchHandlers.push(handler);
  },

  addFailBatchQueryExecutionHandler: function (handler) {
    if (!handler)
      return;

    if (!this.failBatchHandlers)
      this.failBatchHandlers = [];

    this.failBatchHandlers.push(handler);
  },

  addItemToQueue: function (successCallback, failCallback) {
    if (!this.queryQueue)
      this.queryQueue = [];
      this.queryQueue.push({success: successCallback, fail: failCallback});
  },

  _executeQueriesFromQueue: function () {
    if (this.queryQueue &amp;&amp; this.queryQueue.length) {
      var queue = this.queryQueue;

      // we can use our own functions here...
      this.beginBatch();
      jQuery.each(queue, jQuery.proxy(function (key, value) {
        this.executeQueryAsync(value.success, value.fail);
      }, this));
      // we don't need query storage anymore
      delete this.queryQueue;
      this.endBatch();
    }
  }
}

{% endhighlight %}
</div>
So this is it! It's really simple (I've spent a lot more time to write this post than to implement all the logic) but it can help you to avoid a lot of complicated problems.<br />Hope it will be helpful!<br />P.S.: I was using jQuery here to bind functions calls with correct context.<br /><br />Have fun!</div>