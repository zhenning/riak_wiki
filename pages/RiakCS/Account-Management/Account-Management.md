# Account Management

## Creating a User Account
Create a user account by performing an HTTP POST with the username. For example:

```bash
	curl http://localhost:8080/user --data "email=foobar@foobar.com&name=foo%20bar"
```

The JSON response looks like this:

```
{
    "email": "joe@bob.com",
    "display_name": "foobar"
    "key_id": "324ABC0713CD0B420EFC086821BFAE7ED81442C",
    "key_secret": "5BE84D7EEA1AEEAACF070A1982DDA74DA0AA5DA7",
    "name": "foo bar",
    "id":"8d6f05190095117120d4449484f5d87691aa03801cc4914411ab432e6ee0fd6b",
    "buckets": []
}
```

Once the user account exists, you can use the `key_id` and `key_secret` to authenticate requests with Riak CS. To do that, add the key_id and key_secret values to your s3cmd configuration file, which is located by default in the `~/.s3cmd` folder:

* JSON \`key_id\` -> \`~/.s3cmd\` \`access_key\` key_id_value_here
* JSON \`key_secret\` -> \`~/.s3cmd\` \`secret_key\` key_secret_value_here

The canonical id represented by the `id` field can be used as an alternative to an email address for user identification when granting or revoking ACL permissions, for example with the `--acl-grant` or `--acl-revoke` options to `s3cmd setacl`:

* JSON `id` -> `USER_CANONICAL_ID`
