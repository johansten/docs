---
title: Federation
---

Stellar **federation** is a protocol that maps Stellar addresses (email-style addresses) to more information about a given user. It's a way for Stellar client software
to resolve things like `name*yourdomain.com` into account IDs like: `GCCVPYFOHY7ZB7557JKENAX62LUAPLMGIWNZJAFV2MITK6T32V37KEJU`. Stellar addresses provide
an easy way for users to share their payment details, using a syntax that interoperates across different domains and providers.

## Stellar addresses

Stellar addresses are divided into two parts separated by `*`, the username and the domain.

For example:  `jed*stellar.org`
`jed` is the username.
`stellar.org` is the domain.

The domain can be any valid RFC 1035  domain name.
The username is limited to printable UTF-8 with whitespace and the following characters excluded: <*,@> Although of course the domain administrator can place additional restrictions on usernames of its domain.


## Supporting Federation

### Step 1: Create a [stellar.toml](./stellar-toml.md) file

Create a file called stellar.toml and put it `https://www.YOUR_DOMAIN/.well-known/stellar.toml`.

### Step 2: Add federation_url

Add a `FEDERATION_SERVER` section to your stellar.toml file that tells other people the URL of your federation endpoint.

For example: `FEDERATION_SERVER="https://api.yourdomain.com/federation"`

### Step 3: Implement federation url HTTP endpoint

The federation URL specified in your stellar.toml file should accept an HTTP GET request and issue responses of the form detailed below.

## Federation Requests
You can use the federation endpoint to look up an account id if you have a stellar address. You can also do reverse federation and look up a stellar addresses from account ids or transaction ids. This is useful to see who has sent you a payment.

Federation requests have the following form:

`?q=<string to look up>&type=<name,id,txid>`

Supported types:
 - **name**:   Example: `https://api.stellar.org/federation?q=jed*stellar.org&type=name`
 - **id**: *not supported by all federation servers* Will return the federation record of the stellar address associated with the give account id. In some cases this is ambiguous. For instance if a gateway sends transactions on behalf of its users the account id will be of the gateway and the federation server won't be able to resolve the particular user that sent the transaction. In cases like that you may need to use **txid** instead. Example: `https://api.stellar.org/federation?q=GD6WU64OEP5C4LRBH6NK3MHYIA2ADN6K6II6EXPNVUR3ERBXT4AN4ACD&type=id`
 - **txid**: *not supported by all federation servers* Will return the federation record of the sender of the transaction if known by the server. Example: `https://api.stellar.org/federation?q=c1b368c00e9852351361e07cc58c54277e7a6366580044ab152b8db9cd8ec52a
&type=txid`



### Federation Response
The federation server should respond with an appropriate HTTP status code and a JSON response:

```
status: 200
{
    stellar_address: <username*domain.tld>,
    account_id: <account_id>,
    memo_type: <"text", "id" , or "hash"> *optional*
    memo: <memo to attach to any payment. if "hash" type then will be base32 encoded> *optional*
}

or on error,
status: 501
status: 404
etc.
{
   detail: "extra details provided by the federation server"
}
```
