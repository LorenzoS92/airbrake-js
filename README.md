# Airbrake-JS [![Build Status](https://circleci.com/gh/airbrake/airbrake-js.png?circle-token=3c561e089366ed8e0c9ec396a69f9665c1331581)](https://circleci.com/gh/airbrake/airbrake-js)

This is the JavaScript notifier for capturing errors in web browsers and reporting them to [Airbrake](http://airbrake.io).

<img src="http://f.cl.ly/items/443E2J1D2W3x1E1u3j1u/JS-airbrakeman.jpg" width=800px>

## Installation

```
npm install airbrake-js
```

## Setup

Notifier uses [standalone browserify build](http://www.forbeslindesay.co.uk/post/46324645400/standalone-browserify-builds) and can be used with:
- [RequireJS](examples/requirejs/app.js).
- [Global/Window](examples/legacy/app.js).

## Basic Usage

The simplest method is to report errors directly:

    try {
      // This will throw if the document has no head tag
      document.head.insertBefore(document.createElement("style"));
    } catch(err) {
      airbrake.push(err);
      throw err;
    }

Alternatively you can wrap any code which may throw errors using the client's `wrap` method:

    var wrapped = airbrake.wrap(function() {
      // This will throw if the document has no head tag
      document.head.insertBefore(document.createElement("style"));
    });
    wrapped();

## Advanced Usage

The notifier provides a few pieces of functionality to help reduce duplication when you report errors. In order to access this functionality, the full notifier needs to be loaded, not just the shim implementation provided in the embed snippet.

### Default Annotations

It's possible to annotate error notices with all sorts of useful information. Below, the various top-level interface methods are listed, along with their effects.

* `airbrake.setEnvironmentName(string)` Sets the environment name displayed alongside an error report.
* `airbrake.addEnvironment(object)` Merges environment information about the application's environment.
* `airbrake.addParams(object)` Merges params information reported alongside all errors.
* `airbrake.addSession(object)` Merges session information reported alongside all errors.
* `airbrake.addContext(object)` Merges context information reported alongside all errors. This hash is reserved for notifiers and Airbrake backend will ignore unknown keys.

Additionally, much of this information can be added to captured errors at the time they're captured by supplying it in the object being reported.

    try {
      // This will throw if the document has no head tag
      document.head.insertBefore(document.createElement("style"));
    } catch(err) {
      airbrake.push({
        error: err,
        context: { component: 'style', userId: currentUser.id, userName: currentUser.name },
        environment: { navigator_vendor: window.navigator.vendor },
        params:  { search: document.location.search },
        session: { sessionid: sessionid }
      });
      throw err;
    }

### Error object

Instead of exception you can pass error object constructed manually. For example, `window.onerror` handler can look like:

    window.onerror = function(message, file, line) {
      airbrake.push({error: {message: message, fileName: file, lineNumber: line}});
    }

### Source map

In order to enable source map support you have to specify path to the source map file according to the [source map specification](https://docs.google.com/document/d/1U1RGAehQwRypUTovF1KRlpiOFze0b-_2gc6fAH0KY0k/edit#heading=h.lmz475t4mvbx). For example, airbrake.min.js has following line:

    //# sourceMappingURL=airbrake.min.map

### Filtering errors

There may be some errors thrown in your application that you're not interested in sending to Airbrake, such as errors thrown by 3rd-party libraries, or by browser extensions run by your users.

The Airbrake notifier makes it simple to ignore this chaff while still processing legitimate errors. Add filters to the notifier by providing filter functions to `addFilter`.

`addFilter` accepts the entire error notice to be sent to Airbrake, and provides access to the `context`, `environment`, `params`, and `sessions` values submitted with the notice, as well as the single-element `errors` array with its `backtrace` element and associated backtrace lines.

The return value of the filter function determines whether or not the error notice will be submitted.
  * If a falsey value is returned, the notice is suppressed.
  * If a truthy value is returned, the notice may be admissible for submission.

An error notice must pass all provided filters to be submitted.

    // Here we suppress the notice if the top-level `session` key
    // indicates the user is logged in as an admin
    airbrake.addFilter(function(notice) {
      // Suppress reports from admin sessions
      return !notice.session.admin;
    });

### Custom reporters

If you're interested in inspecting the information reported to Airbrake in your own code, you can register your
own error reporter. Note that reporters added this way may be executed out-of-order.

In this example, reported errors are also logged to the console.

    <script>
      airbrake.addReporter(function(notice) {
        console.log(notice);
      });
    </script>

## Testing

### Unit tests

     grunt test

### Karma tests

    grunt karma

## Credits

Airbrake is maintained and funded by [airbrake.io](http://airbrake.io)

Thank you to all [the contributors](https://github.com/airbrake/airbrake-js/contributors).

The names and logos for Airbrake are trademarks of Rackspace Hosting Inc.

# License

Airbrake is Copyright © 2008-2013 Rackspace Hosting Inc. It is free software, and may be redistributed under the terms specified in the MIT-LICENSE file.
