# loopback-component-oauth2

loopback-component-oauth2 provides full integration between [OAuth 2.0](http://tools.ietf.org/html/rfc6749)
and [LoopBack](http://loopback.io). It enables LoopBack applications to function
as an oAuth 2.0 provider to authenticate and authorize client applications and/or
resource owners to access protected API endpoints.

## Usage

OAuth 2.0 defines an authorization framework, allowing an extensible set of
authorization grants to be exchanged for access tokens.  Implementations are
free to choose what grant types to support, by using bundled middleware to
support common types or plugins to support extension types.

#### Create an OAuth Server

Call `createServer()` to create a new OAuth 2.0 server.  This instance exposes
middleware that will be mounted in routes, as well as configuration options.

    var server = oauth2Provider.createServer();

#### Register Grants

A client must obtain permission from a user before it is issued an access token.
This permission is known as a grant, the most common type of which is an
authorization code.

    server.grant(oauth2Provider.grant.code(function(client, redirectURI, user, ares, done) {
      var code = utils.uid(16);

      var ac = new AuthorizationCode(code, client.id, redirectURI, user.id, ares.scope);
      ac.save(function(err) {
        if (err) { return done(err); }
        return done(null, code);
      });
    }));

loopback-component-oauth2 also bundles support for implicit token grants.

#### Register Exchanges

After a client has obtained an authorization grant from the user, that grant can
be exchanged for an access token.

    server.exchange(oauth2Provider.exchange.code(function(client, code, redirectURI, done) {
      AuthorizationCode.findOne(code, function(err, code) {
        if (err) { return done(err); }
        if (client.id !== code.clientId) { return done(null, false); }
        if (redirectURI !== code.redirectUri) { return done(null, false); }

        var token = utils.uid(256);
        var at = new AccessToken(token, code.userId, code.clientId, code.scope);
        at.save(function(err) {
          if (err) { return done(err); }
          return done(null, token);
        });
      });
    }));

loopback-component-oauth2 also bundles support for password and client credential grants.
Additionally, bundled refresh token support allows expired access tokens to be
renewed.

#### Implement Authorization Endpoint

When a client requests authorization, it will redirect the user to an
authorization endpoint.  The server must authenticate the user and obtain
their permission.

    app.get('/dialog/authorize',
      login.ensureLoggedIn(),
      server.authorize(function(clientID, redirectURI, done) {
        Clients.findOne(clientID, function(err, client) {
          if (err) { return done(err); }
          if (!client) { return done(null, false); }
          if (!client.redirectUri != redirectURI) { return done(null, false); }
          return done(null, client, client.redirectURI);
        });
      }),
      function(req, res) {
        res.render('dialog', { transactionID: req.oauth2.transactionID,
                               user: req.user, client: req.oauth2.client });
      });

In this example, [connect-ensure-login](https://github.com/jaredhanson/connect-ensure-login)
middleware is being used to make sure a user is authenticated before
authorization proceeds.  At that point, the application renders a dialog
asking the user to grant access.  The resulting form submission is processed
using `decision` middleware.

     app.post('/dialog/authorize/decision',
       login.ensureLoggedIn(),
       server.decision());
       
Based on the grant type requested by the client, the appropriate grant
module registered above will be invoked to issue an authorization code.

#### Session Serialization

Obtaining the user's authorization involves multiple request/response pairs.
During this time, an OAuth 2.0 transaction will be serialized to the session.
Client serialization functions are registered to customize this process, which
will typically be as simple as serializing the client ID, and finding the client
by ID when deserializing.

    server.serializeClient(function(client, done) {
      return done(null, client.id);
    });

    server.deserializeClient(function(id, done) {
      Clients.findOne(id, function(err, client) {
        if (err) { return done(err); }
        return done(null, client);
      });
    });

#### Implement Token Endpoint

Once a user has approved access, the authorization grant can be exchanged by the
client for an access token.

    app.post('/token',
      passport.authenticate(['basic', 'oauth2-client-password'], { session: false }),
      server.token(),
      server.errorHandler());

[Passport](http://passportjs.org/) strategies are used to authenticate the
client, in this case using either an HTTP Basic authentication header (as
provided by [passport-http](https://github.com/jaredhanson/passport-http)) or
client credentials in the request body (as provided by 
[passport-oauth2-client-password](https://github.com/jaredhanson/passport-oauth2-client-password)).

Based on the grant type issued to the client, the appropriate exchange module
registered above will be invoked to issue an access token.  If an error occurs,
`errorHandler` middleware will format an error response.

#### Implement API Endpoints

Once an access token has been issued, a client will use it to make API requests
on behalf of the user.

    app.get('/api/userinfo', 
      passport.authenticate('bearer', { session: false }),
      function(req, res) {
        res.json(req.user);
      });

In this example, bearer tokens are issued, which are then authenticated using
an HTTP Bearer authentication header (as provided by [passport-http-bearer](https://github.com/jaredhanson/passport-http-bearer))

## Examples

This [example](https://github.com/strongloop/loopback-example-gateway) demonstrates
how to implement an OAuth service provider, complete with protected API access.

## Tests

    $ npm install
    $ npm test
