oauth2orize-jwt-bearer
======================

JSON Web Token (JWT) Bearer Token Exchange Middleware for [OAuth2orize](https://github.com/jaredhanson/oauth2orize).

This module exchanges a JWT for an access token after authenticated, as [defined](http://tools.ietf.org/html/draft-jones-oauth-jwt-bearer-01#section-2.1) by the JSON Web Token (JWT) Bearer Token Profiles for OAuth 2.0 draft.  This module is modeled off of Google's OAuth 2.0 [Server to Server Applications](https://developers.google.com/accounts/docs/OAuth2ServiceAccount).  This module can be used with the [passport-oauth2-jwt-bearer](https://github.com/xtuple/passport-oauth2-jwt-bearer) module to create a JWT OAuth 2.0 exchange scenario server.

## Install

    $ npm install oauth2orize-jwt-bearer

## Usage

#### Register Exchange Middleware

This exchange middleware is used to by clients to request an access token by using a JSON Web Token (JWT) generated by the client and verified by a Public Key stored on the OAuth 2.0 server.  The exchange requires a verify callback, which accepts the client, JWT data and signature, then calls done providing a access token. 

##### Key Generation Tips
generate private key
openssl genrsa -out private.pem 1024 

abstract public key
openssl rsa -in private.pem -out public.pem -outform PEM -pubout 

sign the data
signing data: echo -n "data-to-sign" | openssl dgst -RSA-SHA256 -sign private.pem > signed 

convert the signed file (binary) into base64 to be sent.
base64 signed

    var jwtBearer = require('oauth2orize-jwt-bearer').Exchange;

    server.exchange('urn:ietf:params:oauth:grant-type:jwt-bearer', jwtBearer(function(client, data, signature, done) {
     var crypto = require('crypto')
	   , fs = require('fs') //load file system so you can grab the public key to read.
       , pub = fs.readFileSync('/path/to/public.pem').toString() //load PEM format public key as string, should be clients public key
       , verifier = crypto.createVerify("RSA-SHA256");

	 //verifier.update takes in a string of the data that is encrypted in the signature  
     verifier.update(JSON.stringify(data));
    
     if (verifier.verify(pub, signature, 'base64')) {
	   //base64url decode data 
	   var b64string = data;
	   var buf = new Buffer(b64string, 'base64').toString('ascii');
	   
       // TODO - verify client_id, scope and expiration are valid from the buf variable above

       AccessToken.create(client, scope, function(err, accessToken) {
         if (err) { return done(err); }
         done(null, accessToken);
       });
     }
    }));

## Tests

    $ npm install --dev
    $ make test

## Credits

  - [bendiy](http://github.com/bendiy)

## License

[The MIT License](http://opensource.org/licenses/MIT)

Copyright (c) 2012-2013 xTuple <[http://www.xtuple.com/](http://www.xtuple.com/)>