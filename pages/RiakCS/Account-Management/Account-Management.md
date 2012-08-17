# Account Management

## Creating a User Account
Create a user account by performing an HTTP POST or PUT with a unique email address and username. For example:

```bash
curl -H 'Content-Type: application/json' -X POST http://localhost:8080/user --data '{"email":"foobar@example.com", "name":"foo bar"}'
```

The submitted user document may be either JSON or XML, but the type should match the value of the Content-Type header used.

Here are some examples for JSON and XML input formats.

JSON:

```json
{
  "email" : "foobar@example.com",
  "name" : "foo bar"
}
```

XML:

```xml
<User>
  <Email>foobar@example.com</Email>
  <Name>foo bar</Name>
</User>
```

The response will be in JSON or XML, and resembles the following examples.

JSON:

```json
{
    "email": "foobar@example.com",
    "display_name": "foobar"
    "key_id": "324ABC0713CD0B420EFC086821BFAE7ED81442C",
    "key_secret": "5BE84D7EEA1AEEAACF070A1982DDA74DA0AA5DA7",
    "name": "foo bar",
    "id":"8d6f05190095117120d4449484f5d87691aa03801cc4914411ab432e6ee0fd6b",
    "buckets": []
}
```

XML:

```xml
<User>
  <Email>foobar@example.com</Email>
  <DisplayName>foobar</DisplayName>
  <KeyId>324ABC0713CD0B420EFC086821BFAE7ED81442C</KeyId>
  <KeySecret>5BE84D7EEA1AEEAACF070A1982DDA74DA0AA5DA7</KeySecret>
  <Name>foo bar</Name>
  <Id>8d6f05190095117120d4449484f5d87691aa03801cc4914411ab432e6ee0fd6b</Id>
  <Buckets></Buckets>
</User>
```

Once the user account exists, you can use the `key_id` and `key_secret` to authenticate requests with Riak CS. To do that, add the key_id and key_secret values to your s3cmd configuration file, which is located by default in the `~/.s3cmd` folder:

* JSON \`key_id\` -> \`~/.s3cmd\` \`access_key\` key_id_value_here
* JSON \`key_secret\` -> \`~/.s3cmd\` \`secret_key\` key_secret_value_here

The canonical id represented by the `id` field can be used as an alternative to an email address for user identification when granting or revoking ACL permissions, for example with the `--acl-grant` or `--acl-revoke` options to `s3cmd setacl`:

* JSON `id` -> `USER_CANONICAL_ID`

<div class="note"><div class="title">Note</div>
By default, only the admin user may create new user accounts. If you need to create a user account without authenticating yourself, you must set <tt>{anonymous_user_creation, true}</tt> in the Riak CS <tt>app.config</tt>.
</div>