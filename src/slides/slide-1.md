class: center, middle

# Vault Overview

---

# What it is:

> Vault secures, stores, and tightly controls access to tokens, passwords,
> certificates, API keys, and other secrets in modern computing. Vault handles
> leasing, key revocation, key rolling, and auditing. Through a unified API,
> users can access an encrypted Key/Value store and network
> encryption-as-a-service, or generate AWS IAM/STS credentials, SQL/NoSQL
> databases, X.509 certificates, SSH credentials, and more. 
>
> https://www.vaultproject.io/

---

# Vault Components

- Vault server:
        - connects to the secret backend(s)
        - manages authentication and audit
        - seals/unseals vault

- Vault CLI client:
        - same binary, incidentally
        - environment variable for server URI
        - management commands
        - retrieve and manage secrets
---

# Vault Components

- HTTP API
        - same capabilities as CLI client
        - CLI uses this API

- Libraries
        - Go, Ruby (official)
        - Ansible, C#, Clojure, Elixr, Java, Kotlin, Node.js
          PHP, Python, Scala (community supported)

---

# Vault Architecture

[<img src="https://www.vaultproject.io/assets/images/layers-368ccce4.png" style="width: 75%; height: 75%"/>](https://www.vaultproject.io/docs/internals/architecture.html)

> The storage backend is untrusted and is used to durably store encrypted data.
> When the Vault server is started, it must be provided with a storage backend
> so that data is available across restarts. The HTTP API similarly must be
> started by the Vault server on start so that clients can interact with it.

---

# Introducing Svalbard

> _Svalbard_ is the name I'd chosen for this implementation of Vault

[<img src="https://upload.wikimedia.org/wikipedia/commons/6/6b/Svalbard_seed_vault_IMG_8894.JPG" style="width: 50%; height: 50%"/>](https://www.regjeringen.no/en/topics/food-fisheries-and-agriculture/landbruk/svalbard-global-seed-vault/id462220/)

> Way up north, in the permafrost, 1300 kilometers beyond the Arctic Circle, is
> the world's largest secure seed storage, opened by the Norwegian Government
> in February 2008. From all across the globe, crates of seeds are sent here
> for safe and secure long-term storage in cold and dry rock vaults.

---

# Svalbard's Other Bits

## The Storage Backend

[consul](https://www.consul.io/) is a multi-datacenter capbable system
providing service discovery, failure detection, and key/value storage.  Vault
uses many of Consul's features to provide a highly-available service

## A Certificate Authority

Consul relies on certificates to provide secure communication between nodes as
well as providing an authentication backend for host or application
authentication.

???

* A CA is required to ensure that specific _hosts_ can be authenticated.  The
wildcard cert provides no such assurances.
* The cert needs to provide multiple DNS names and the IP address

---

# Vault Operations

## Seal/Unseal

- Secrets are encrypted- unsealing makes secrets available to the Vault server.
- During initialization of the vault, a master key is generated and split into
  5 shards
- 3 shards are required to unseal.
  - see [Shamir's Secret Sharing](https://en.wikipedia.org/wiki/Shamir%27s_Secret_Sharing)
- A root token is also created- this should be destroyed after configuring additional roles and tokens assigned to administrators

???

```
   bin/vault seal
   bin/vault unseal
```

---

# Vault Operations

 - Authenticate (login) via connected authentication backend
 - Receive a token which is attached to an access policy
 - Provide token with a request for information on a path
 - Assuming access policy on token allows, return secret

<img src="img/vault-get-secret.svg"/>
???
All operations (even the `vault` command) use HTTP API

Access is driven from tokens
 - enables RBAC to secrets &c
 - enables expiring and revoking access to secrets
 - since tokens do expire, refreshing is a necessary part of the workflow

---

# Vault Operations

## Authentication Backends

  - Many Options:
    - Tokens \*
    - AppRole \*
    - GitHub
    - LDAP/AD
    - Username/Password

\* currently active in Svalbard

???

* LDAP/AD for carbon-based life forms
* AppRole and Tokens for software and services

---

# Vault Operations

## AppRole Authentication

> [... allows machines and services (apps) to authenticate with Vault via a series of administratively defined roles: AppRoles (https://www.vaultproject.io/docs/auth/approle.html)](https://www.vaultproject.io/docs/auth/approle.html)

 - A role is created that has an access policy attached to it
 - A secret ID is then created using the role ID
 - The secret and role IDs are then used by the client to authenticate
 - Vault provides a token with the role's policy to the client which is then
   used for accessing secrets
 - Access to a role can also be restricted to a CIDR range

<img src="img/approle-auth.svg"/>

???

Adding `bound_cidr_list` and setting `bind_secret_id` to false allows distribution to IP ranges using only the role ID

---

# Vault Operations

## Token Authentication

* Token authentication is at the root of every authentication backend (i.e. other backends create tokens using this backend).
* A `root` token is initially created- "best practice" recommends against keeping this token
* Tokens are heirarchical (i.e. revocation of a parent removes children)
* Distribution of tokens still a challenge

???

* Tokens created with this backend need to be handled like traditional secrets
* Can be used by external services to create tokens dynamically



---

# Vault Operations

## Access Policies

A policy is the composition of a path and the capabililities on that path assigned to a token. Tokens can have multiple policies.

```
path "secret/*" {
  policy = "write"
}

path "secret/foo" {
  policy = "read"
}

path "auth/token/lookup-self" {
  policy = "read"
}
```

---

# Using Vault - AppRole

I've configured a policy and role both named "core" which could be used for
storing essential, but not particularly sensitive, information (e.g. DNS
servers and other domain-local information)

```
root@sv-vault-srv01# vault read auth/approle/role/core        
Key                     Value
---                     -----
bind_secret_id          false
bound_cidr_list         140.107.0.0/16
period                  0
policies                [core default]
secret_id_num_uses      0
secret_id_ttl           0
token_max_ttl           0
token_ttl               0
```

---

# Using Vault - AppRole

This policy creates read access for paths `secret/core` and anything created
underneath that point

```
root@sv-vault-srv01# vault policies core
path "secret/core*" {
  capabilities = ["read"]
}
path "secret/core/*" {
  capabilities = ["read"]
}
```

`default` has a bunch of required privileges:

```
root@sv-vault-srv01# vault policies default
# Allow tokens to look up their own properties
path "auth/token/lookup-self" {
  capabilities = ["read"]
}
# Allow tokens to renew themselves
path "auth/token/renew-self" {
  capabilities = ["update"]
}
```
---

# Using Vault - AppRole

Managing secrets in this path requires more security.  Here I've configured an
admin role requiring the generation of a secret ID and that the login comes
from a particular host

```
root@sv-vault-srv01# vault read auth/approle/role/core-admin
Key                     Value
---                     -----
bind_secret_id          true
bound_cidr_list         140.107.72.78/32
period                  0
policies                [core-admin default]
secret_id_num_uses      0
secret_id_ttl           30m
token_max_ttl           0
token_ttl               0
```

---

# Using Vault - AppRole

The policy for this role has considerably more privileges:

```
root@sv-vault-srv01# vault policies core-admin
path "secret/core*" {
  capabilities = ["list","update","delete","create","read"]
}
path "secret/core/*" {
  capabilities = ["list","update","delete","create","read"]
}
```

---

# Using Vault - AppRole

The `role_id` is set to the friendly name of "core".  The `core` role doesn't
require a secret ID:

```
talos$ curl -X POST -d '{"role_id": "core"}'\
> ${VAULT_ADDR}/v1/auth/approle/login |prettyjson
{
    "auth": {
        "accessor": "1efc2cae-f0d8-6664-77e4-926f280f2bf2",
        "client_token": "08a293df-be19-2c7b-2043-2f1ead0bf57e",
        "lease_duration": 2764800,
        "metadata": null,
        "policies": [
            "core",
            "default"
        ],
        "renewable": true
    },
    "data": null,
    ...
}
```
???

* `prettyjson` is an alias for `python -m json.tool`
* the `role_id` is usually a guuid, can be set using `vault write auth/approle/role/core role_id=<some string>`

---

# Using Vault - AppRole

`client_token` is then used to get a secret:

```
mrg@talos$ curl -X GET -H "X-Vault-Token:$VAULT_TOKEN" \
>   ${VAULT_ADDR}/v1/secret/core/dallas |prettyjson
{
    "auth": null,
    "data": {
        "who shot jr": "hoffa did it"
    },
    "lease_duration": 2764800,
    "lease_id": "",
    "renewable": false,
    "request_id": "966a7df7-bfde-a468-ec98-ac88ddf1765a",
    "warnings": null,
    "wrap_info": null
}
```

---

# Using Vault - AppRole

This token can't update the secret:

```
mrg@talos$ curl -X POST -d '{"hoovers dress size":"12"}' \
> -H "X-Vault-Token:$VAULT_TOKEN" ${VAULT_ADDR}/v1/secret/core | prettyjson
{
    "errors": [
        "permission denied"
    ]
}
```

We need to get a secret-id for the _core-admin_ role which is then used to log in:

```
mrg@talos$ curl -X POST -d '{"role_id":"core-admin"}' \
> -H "X-Vault-Token:${VAULT_TOKEN}" \
> ${VAULT_ADDR}/v1/auth/approle/role/core-admin/secret-id |prettyjson
{
    "auth": null,
    "data": {
        "secret_id": "008bf2cf-6551-05a4-7f73-3ab9fe2cf0e7",
        "secret_id_accessor": "b5f174ec-6390-faf3-3761-df5ae1baa557"
    },
        ...

```

???

* Typically we'd use a username/password type scheme as this role is for humans- logging in with the username and password

---

# Using Vault - AppRole

```
mrg@talos$ curl -X POST \
> -d '{"role_id":"core-admin","secret_id":"73b1cce6-448e-dff0-bd24-2a4783fe221d"}'\
> ${VAULT_ADDR}/v1/auth/approle/login |prettyjson
{
    "auth": {
        "accessor": "bc4677df-19d7-3d0d-877d-16b7de7a90ce",
        "client_token": "76a458cc-7be8-1613-00bd-552967b6fd33",
        "lease_duration": 2764800,
        "metadata": {},
        "policies": [
            "core-admin",
            "default"
        ],
        "renewable": true
    },
    ...
```

---

# Using Vault - AppRole

The token can then be used to write to those paths:

```
mrg@talos$  export VAULT_TOKEN=76a458cc-7be8-1613-00bd-552967b6fd33
mrg@talos$ curl -X POST -d '{"hoovers dress size":"10"}' \
> -H "X-Vault-Token:$VAULT_TOKEN" ${VAULT_ADDR}/v1/secret/core
mrg@talos$ curl -X GET -H "X-Vault-Token:$VAULT_TOKEN" \
${VAULT_ADDR}/v1/secret/core |prettyjson 
{
    "auth": null,
    "data": {
        "hoovers dress size": "10"
    },
    "lease_duration": 2764800,
    "lease_id": "",
    "renewable": false,
    "request_id": "17ef00b9-f7d4-a4f2-b19c-196430169e41",
    "warnings": null,
    "wrap_info": null
}
```

???

* No, the leading space in front of the `export` command isn't a typo- it's there to prevent the token from being stored in the history file

---
