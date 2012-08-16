# Authentication
Currently, the only authentication scheme available to use with Riak CS is the S3 authentication scheme. A signature is calculated using several elements from each request including the user's *key_id* and *key_secret*.

This signature is included in the *Authorization* header of the request. Once a request is received by the server, the server also calculates the signature for the request and compares the result with the signature presented in then *Authorization* header. If they match then the request is authenticated; otherwise, the authentication fails.

Full details on the S3 authentication scheme are available in Amazon's [[Signing and Authenticating REST Requests|http://docs.amazonwebservices.com/AmazonS3/latest/dev/RESTAuthentication.html]] documentation.

If **auth_bypass** is set to true in `app.config`, the system ignores the Signature portion of the authorization header described below:

```
Authorization: AWS AWSAccessKeyId:Signature
```