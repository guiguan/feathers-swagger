# feathers-swagger

[![Build Status](https://travis-ci.org/feathersjs/feathers-swagger.png?branch=master)](https://travis-ci.org/feathersjs/feathers-swagger)
[![Code Climate](https://codeclimate.com/github/feathersjs/feathers-swagger/badges/gpa.svg)](https://codeclimate.com/github/feathersjs/feathers-swagger)
[![Test Coverage](https://codeclimate.com/github/feathersjs/feathers-swagger/badges/coverage.svg)](https://codeclimate.com/github/feathersjs/feathers-swagger/coverage)
[![Dependency Status](https://img.shields.io/david/feathersjs/feathers-swagger.svg?style=flat-square)](https://david-dm.org/feathersjs/feathers-swagger)
[![Download Status](https://img.shields.io/npm/dm/feathers-swagger.svg?style=flat-square)](https://www.npmjs.com/package/feathers-swagger)

> Add documentation to your Featherjs services and show them in the Swagger ui. Jsdoc documentation comment is supported.

## Installation

```
npm install feathers-swagger --save
```

## Examples

> npm install feathers feathers-rest feathers-swagger body-parser

### Basic example

Here's an example of a Feathers server that uses `feathers-swagger`.

```js
const feathers = require('feathers');
const rest = require('feathers-rest');
const bodyParser = require('body-parser');
const swagger = require('feathers-swagger');

class MessageService {
  constructor(options = {}) {
    this.options = options;
    // traditional way, takes higher precedence than jsdoc
    this.docs = {
      create: {
        description: 'Create a new message',
        parameters: [{
          in: 'query',
          name: 'url',
          description: 'Callback URL',
          type: 'string'
        }, {
          in: 'body',
          name: 'message',
          description: 'Message to send',
          type: 'string'
        }, {
          in: 'body',
          name: 'includePics',
          description: 'Whether to include pics in the message',
          type: 'boolean'
        }]
      },
      remove: {
        description: 'Remove a message',
        parameters: [{
          in: 'path',
          name: 'id',
          description: 'Message ID',
          type: 'number'
        }]
      }
    };
  }

  // new way of documenting APIs using Jsdoc. Recommended!

  /**
   * Create a new message
   *
   * @param {String} params.url - [query] callback URL
   * @param {String} params.message - [body] message to send
   * @param {Boolean} params.includePics - whether to include pics in the message [body]
   */
  create(data, params) {
    ...
  }

  /**
   * Remove a message
   *
   * @param {Number} params.id - [path] message ID
   */
  remove(id, params) {
    ...
  }
}

// Make sure to decorate your service class
MessageService = swagger.applyJsdoc(MessageService);

const app = feathers()
  .use(bodyParser.json())
  .use(bodyParser.urlencoded({ extended: true }))
  .configure(rest())
  .configure(swagger({
    docsPath: '/docs',
    info: {
      title: 'A test',
      description: 'A description'
    }
  }))
  .use('/messages', new MessageService());

app.listen(3030);
```

Go to `localhost:3030/docs` to see the Swagger JSON documentation.

### Example with UI

The `uiIndex` option allows to set a [Swagger UI](http://swagger.io/swagger-ui/) index file which will host the UI at `docsPath`.

```js
const path = require('path');
const feathers = require('feathers');
const rest = require('feathers-rest');
const bodyParser = require('body-parser');
const swagger = require('feathers-swagger');

// same as before
...

const app = feathers()
  .use(bodyParser.json())
  .use(bodyParser.urlencoded({ extended: true }))
  .configure(rest())
  .configure(swagger({
    docsPath: '/docs',
    uiIndex: path.join(__dirname, 'docs.html'),
    info: {
      title: 'A test',
      description: 'A description'
    }
  }))
  .use('/messages', new MessageService());

app.listen(3030);
```

Create a `docs.html` page like this:

```html
<!DOCTYPE html>
<html>
<head>
  <title>Swagger UI</title>
  <link rel="icon" type="image/png" href="images/favicon-32x32.png" sizes="32x32" />
  <link rel="icon" type="image/png" href="images/favicon-16x16.png" sizes="16x16" />
  <link href='css/typography.css' media='screen' rel='stylesheet' type='text/css'/>
  <link href='css/reset.css' media='screen' rel='stylesheet' type='text/css'/>
  <link href='css/screen.css' media='screen' rel='stylesheet' type='text/css'/>
  <link href='css/reset.css' media='print' rel='stylesheet' type='text/css'/>
  <link href='css/print.css' media='print' rel='stylesheet' type='text/css'/>

  <script src='lib/object-assign-pollyfill.js' type='text/javascript'></script>
  <script src='lib/jquery-1.8.0.min.js' type='text/javascript'></script>
  <script src='lib/jquery.slideto.min.js' type='text/javascript'></script>
  <script src='lib/jquery.wiggle.min.js' type='text/javascript'></script>
  <script src='lib/jquery.ba-bbq.min.js' type='text/javascript'></script>
  <script src='lib/handlebars-4.0.5.js' type='text/javascript'></script>
  <script src='lib/lodash.min.js' type='text/javascript'></script>
  <script src='lib/backbone-min.js' type='text/javascript'></script>
  <script src='swagger-ui.js' type='text/javascript'></script>
  <script src='lib/highlight.9.1.0.pack.js' type='text/javascript'></script>
  <script src='lib/highlight.9.1.0.pack_extended.js' type='text/javascript'></script>
  <script src='lib/jsoneditor.min.js' type='text/javascript'></script>
  <script src='lib/marked.js' type='text/javascript'></script>
  <script src='lib/swagger-oauth.js' type='text/javascript'></script>
  <script type="text/javascript">
    $(function () {

      var url = "/docs";

      hljs.configure({
        highlightSizeThreshold: 5000
      });

      // Pre load translate...
      if(window.SwaggerTranslator) {
        window.SwaggerTranslator.translate();
      }
      window.swaggerUi = new SwaggerUi({
      url: url,
      dom_id: "swagger-ui-container",
      supportedSubmitMethods: ['get', 'post', 'put', 'delete', 'patch'],
      onComplete: function(swaggerApi, swaggerUi){
        if(typeof initOAuth == "function") {
          initOAuth({
            clientId: "your-client-id",
            clientSecret: "your-client-secret-if-required",
            realm: "your-realms",
            appName: "your-app-name",
            scopeSeparator: " ",
            additionalQueryStringParams: {}
          });
        }

        if(window.SwaggerTranslator) {
          window.SwaggerTranslator.translate();
        }
      },
      onFailure: function(data) {
        log("Unable to Load SwaggerUI");
      },
      docExpansion: "none",
      jsonEditor: false,
      defaultModelRendering: 'schema',
      showRequestHeaders: false
    });

    $('#input_apiKey').change(function() {
      var key = $('#input_apiKey')[0].value;
      log("key: " + key);
      if(key && key.trim() != "") {
        log("added key " + key);
        window.authorizations.add("key", new ApiKeyAuthorization("api_key", key, "query"));
      }
    })
    window.swaggerUi.load();
  });

  </script>
</head>


<body class="swagger-section">
<div id='header'>
  <div class="swagger-ui-wrap">
    <a id="logo" href="http://swagger.io"><img class="logo__img" alt="swagger" height="30" width="30" src="images/logo_small.png" /><span class="logo__title">swagger</span></a>
  </div>
</div>

<div id="message-bar" class="swagger-ui-wrap" data-sw-translate>&nbsp;</div>
<div id="swagger-ui-container" class="swagger-ui-wrap"></div>
</body>

</html>
```

Now [localhost:3030/docs/](http://localhost:3030/docs/) will show the documentation in the browser using the Swagger UI.

## License

Copyright (c) 2017

Licensed under the [MIT license](LICENSE).
