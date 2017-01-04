# Other Authentications - Username/Password

A username and password pair are used to log in:

```
mrg@talos$ curl $VAULT_ADDR/v1/auth/userpass/login/mrg \
> -d '{"password":"letmein"}'  |prettyjson 
{
    "auth": {
        "accessor": "21168915-6d5e-00ee-2ca9-804c48d25999",
        "client_token": "e4638b3c-f49b-e701-c8f6-3c1cd6861afe",
        "lease_duration": 86700,
        "metadata": {
            "username": "mrg"
        },
        "policies": [
            "core-admin",
            "default"
        ],
        "renewable": true
    },
    ...
```

???

* note that this token has the _core-admin_ policy attached- it can be used to update tokens.

---

# Other Authentications - Username/Password

Alternatively, using the Vault CLI:

```
mrg@sv-vault-srv01:/opt/svalbard/vault$ vault auth -method=userpass \
> username=mrg \
> password=letmein
==> WARNING: VAULT_TOKEN environment variable set!

  The environment variable takes precedence over the value
  set by the auth command. Either update the value of the
  environment variable or unset it to use the new token.

Successfully authenticated! You are now logged in.
The token below is already saved in the session. You do not
need to "vault auth" again with the token.
token: 2831040b-d816-41ad-7f88-2156f739cf36
token_duration: 86404
token_policies: [aws-all-access core-admin default]
```

---

# Other Authentications - TLS

Store a CA certificate in `auth/cert/certs/<name>`.  Any certificate that can
be verified by this CA can be used to login to this endpoint

```

mrg@talos$ curl --key /var/tmp/sv-vault-srv01.key --cert /var/tmp/sv-vault-srv01.pem -X POST ${VAULT_ADDR}/v1/auth/cert/login |prettyjson 
{
    "auth": {
        "accessor": "4f9345c3-9f14-6584-de90-3a4a333d7af0",
        "client_token": "5372a885-2af8-4b8f-f856-b169d7227cb4",
        "lease_duration": 2764800,
        "metadata": {
            "authority_key_id": "99:0f:66:ea:8b:61:da:45:c5:f0:ff:c0:7b:f9:4f:14:b7:85:e8:0a",
            "cert_name": "svalbard",
            "common_name": "sv-vault-srv01.fhcrc.org",
            "subject_key_id": "f9:e3:84:f9:31:2f:91:43:e9:c7:de:43:e5:36:3d:b7:6b:0d:df:0c"
        },
        "policies": [
            "core",
            "default"
        ],
```

---

# Other Authentications - and the rest

## LDAP

 - Bind Vault to an LDAP/Active Directory server.
 - Map Users and Groups to policy lists

## GitHub

 - Personal access tokens are used for login
 - GitHub teams are mapped to policy lists

## Token

 - Create a token attached to a policy
 - Pass out the token to the service/server/etc.

---

# Other Authentications - and the rest

## MFA

  - Adds MFA to _userpass_ and _ldap/ad_ authentication backends
  - Only [duo](https://duo.com/) supported at the moment

## AWS EC2

  - an "introduction mechanism" using AWS EC2 metadata for
    authorizing instances
  - Vault verifies signature on the encrypted & signed metadata
  - Highly complicated and configurable dance!

---
