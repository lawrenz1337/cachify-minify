# Cachify-minify #
``cachify-minify`` makes having proper browser cache and HTTP caching. It creates a minified pack and checks for its integrity every time user requests for it. It rebuild pack each time one of included files changes.

It is a set of middleware and view helper functions for the Node.js express framework.

## Credits ##
Hey, there! This module is made from combining [connect-cachify](https://www.npmjs.com/package/connect-cachify) and [node-minify](https://www.npmjs.com/package/node-minify)
Its purpose is to create a minified package from input files and rebuild package everytime one of included files changes without app restart. It no longer supports caching of css files because ``node-minify`` minifies only JS files. I hope you'll like it :)

## Installation ##

    npm install cachify-minify

## How to Use ##

    var express = require('express');
    var cachify = require('cachify-minify');
    var app = express();

## Middleware ##
    var assets = {
        "/js/main.min.js": [
          '/js/lib/jquery.js',
          '/js/magick.js',
          '/js/laughter.js'
        ]
    };

    app.use(cachify.setup(assets, {
      root: __dirname
    }));

``setup`` takes two parameters: assets and options. Assets is an associative
array where the keys are your production urls, and the value is a list of
development urls that produce the same asset.

We'll discussion options in a section below.

Cachify middleware is now enabled. Let's look at this after hooking up the view
helpers.

Note: You **must** put ``cachify.setup`` before static or other connect
middleware which works with these same requests.

## In an EJS template

    ...
    <head>
      <title>Dashboard: Hamsters of North America</title>
    </head>
    <body>
    ...
      <%- cachify_js('/js/main.min.js') %>
    </body>
    ...

## In a Jade template

    ...
    title= Dashboard: Hamsters of North America
    meta(charset='utf-8')
    ...
    body
    ...
      | !{cachify_js('/js/main.min.js')}
      block scripts


The middleware makes caching transparent. A request for
``/fa6d51a13a245a90aeb48eeca0e52396/js/main.min.js`` will have the req.url
rewritten to ``/js/main.min.js``, so that other middleware will work properly.

The middleware sets the cache expiration headers to the Mayan Apocalypse, and
does it's best to ensure browsers won't request that version of
``/js/main.min.js`` again.

A goal is for this module to work well with other connect compilers, such as
[LESS](http://lesscss.org/) or
[connect-assets](https://github.com/TrevorBurnham/connect-assets).

## Options ##
The following are optional config for ``cachify.setup``

* root - Path where static assets are stored on disk. Same value as you'd pass
to the ``static`` middleware.

## Magick ##
So how does cachify work?

When you cachify a url, it adds an MD5 hash of the file's contents into the URL
it generates:

    http://example.com/cbcb1e865e61c08a68a4e0bfa293e806/main.js

Incoming requests are checked for this MD5 hash. If present and if we' know
about the resource (either via options or the file exists on disk), then the
request path is rewritten back to ``/main.js``, so that another route can
process the request.

These requests are served up with expires headers that are very long lived, so a user's browser will only request them once.

Cachify **doesn't** attempt to **find an older version** of your resource,
if the MD5 has was for an older file.

## Debugging ##
To debug cachify's hashed url behavior, pass in the following parameter in
your options block:

    setup({ debug: true, ...});
