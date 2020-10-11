---
layout: post
title: "Monitoring the Progress of an HTTP Request in JavaScript"
date: 2020-04-10 17:00:00 +0800
tags: [javascript]
navigation: true
---

When submitting requests to a server, you may want to know its progress, for example when uploading large files which can time.
Several approaches can be taken for monitoring both the upload and download progress, with this post showing two different methods.
Towards the end of the post, it will show how to use these different methods with jQuery, and discuss further steps that can be taken.

## XMLHttpRequest
To send data from a client to a server, an HTTP request needs to be created and sent.
To create an HTTP request in JavaScript, the XMLHttpRequest (XHR) API can be used.
An example of creating a new instance of XHR is shown below:

```javascript
let xhr = new XMLHttpRequest();
```

> More information about the XMLHttpRequest API can be found over at [MDN web docs](https://developer.mozilla.org/en-US/docs/Web/API/XMLHttpRequest/Using_XMLHttpRequest).

## Monitoring progress
To find out the progress of an XHR request, including both upload and download progress, we need to listen for an **event**. In this case, the [ProgressEvent](https://developer.mozilla.org/en-US/docs/Web/API/ProgressEvent) is required.

### ProgressEvent
The ProgressEvent object has three properties that are useful in finding out the current progress:
* lengthComputable - states whether the progress of a request can be measured
* loaded - the amount of data sent
* total - the total amount of data that needs to be sent

These three properties can be combined to calculate the progress, as shown below:

```javascript
if (event.lengthComputable) {
    percentComplete = Math.round((event.loaded / event.total) * 100);
}
```

### Download progress
The following sections all discuss how to capture and use information relating to the progress of downloading from a server.
Two different approaches can be taken, with both discussed below.

#### EventTarget
To listen for a ProgressEvent and get access to these properties, listeners can be attached to the XHR object.
Attaching listeners to an object can be achieved through the [EventTarget API](https://developer.mozilla.org/en-US/docs/Web/API/EventTarget), as XMLHttpRequest inherits properties from EventTarget.
This means XHR has access to the **addEventListener** method, which can listen for a specific event.
An example of how to attach an event listener for the ProgressEvent using the addEventListener method is shown below:

```javascript
xhr.addEventListener("progress", function (event) {
    ...
});
```

> The event variable will be an instance of ProgressEvent with the properties discussed above and, therefore, the calculation to compute the percentage completed can be used here.

#### XMLHttpRequestEventTarget
While using EventTarget to attach a listener is fine, XHR also inherits from [XMLHttpRequestEventTarget](https://developer.mozilla.org/en-US/docs/Web/API/XMLHttpRequestEventTarget), which happens to have an [onprogress property](https://developer.mozilla.org/en-US/docs/Web/API/XMLHttpRequestEventTarget/onprogress).
This property is a callback function that gets called with ProgressEvent as an argument.
An example of how to set the onprogress property is shown below:

```javascript
xhr.onprogress = function (event) {
    ...
};
```

### Upload progress
Capturing information relating to the upload progress is similar to the methods used for download progress.
To access the upload progress, XHR has an [upload property](https://developer.mozilla.org/en-US/docs/Web/API/XMLHttpRequest/upload).

Again, we can either attach a listener to the upload property of XHR or the onprogress property of upload can be set with a callback function. Both are shown below:

```javascript
// Attach a listener
xhr.upload.addEventListener("progress", function (event) {
    ...
});

// Callback function
xhr.upload.onprogress = function (event) {
    ...
};
```

### jQuery AJAX request
Now that monitoring the upload and download progress is known, integrating this with a [jQuery AJAX](https://api.jquery.com/jquery.ajax/#jQuery-ajax-url-settings) request is easy.
To attach an event listener or set the onprogress property, the instance of XHR that is used by jQuery needs to be adjusted.
This can be achieved using the xhr property of $.ajax(), where the creation of XHR is overridden with the function provided, as shown below:

```javascript
// Attach a listener
$.ajax({
    xhr: function () {
        let xhr = new XMLHttpRequest();

        xhr.addEventListener("progress", function (event) {
            ...
        });

        xhr.upload.addEventListener("progress", function (event) {
            ...
        });

        return xhr;
    },
});

// Callback function
$.ajax({
    xhr: function () {
        let xhr = new XMLHttpRequest();

        xhr.onprogress = function (event) {
            ...
        };

        xhr.upload.onprogress = function (event) {
            ...
        };

        return xhr;
    },
});
```

## Summary
In summary, this post looked at how to monitor the progress of an HTTP request between a client and a server, including both uploading and downloading.
The ProgressEvent object contains the information necessary for calculating the progress of a request, which can be gained by either attaching a listener for this event or by setting the onprogress property of the XMLHttpRequest object.
Additionally, utilising these two methods with a jQuery AJAX request was shown.

An alternative method to the ones shown above is adding progress promises to $.ajax(), with a project implementing this by [likerRr](https://github.com/likerRr/jq-ajax-progress).
Furthermore, I added an example of how to display a request's progress as a progress bar in a [GitHub Gist](https://gist.github.com/jonathanstaniforth/4e90125eb324c19bafc2e6f46db444a2).
