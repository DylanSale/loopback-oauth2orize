# loopback-component-oauth2


Forked original repo at [2.0.0-rc3](https://github.com/strongloop/loopback-component-oauth2/commit/2422c47d9a80928c895200cafc170dc86b184fe0)
due to licence change to Strongloop.

The LoopBack oAuth 2.0 component provides full integration between [OAuth 2.0](http://tools.ietf.org/html/rfc6749)
and [LoopBack](http://loopback.io). It enables LoopBack applications to function
as an oAuth 2.0 provider to authenticate and authorize client applications and/or
resource owners (i.e. users) to access protected API endpoints.

The oAuth 2.0 protocol implementation is based on [oauth2orize](https://github.com/jaredhanson/oauth2orize)
and [passport](http://passportjs.org/).

See [LoopBack Documentation - OAuth 2.0 Component](http://docs.strongloop.com/display/LB/OAuth+2.0) for more information.

## Install

Install the component from this fork:

```
$ npm install https://github.com/ticadia/loopback-oauth2orize/tarball/v2.2.0
```

Or add to package.json directly:

```
"loopback-oauth2orize": "git://github.com/ticadia/loopback-oauth2orize.git"
```

## Use

Use in an application as follows:

```js
var oauth2 = require('loopback-component-oauth2');

var options = {
  dataSource: app.dataSources.db, // Data source for oAuth2 metadata persistence
  loginPage: '/login', // The login page url
  loginPath: '/login' // The login form processing url
};

oauth2.oAuth2Provider(
  app, // The app instance
  options // The options
);
```

The app instance will be used to set up middleware and routes. The data source
provides persistence for the oAuth 2.0 metadata models.

For more information, see [OAuth 2.0](http://docs.strongloop.com/display/LB/OAuth+2.0) LoopBack component official documentation.
