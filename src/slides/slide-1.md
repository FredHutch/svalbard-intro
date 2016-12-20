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

---

# Vault Operations

## Seal/Unseal

- Secrets are encrypted- unsealing makes secrets available to the Vault server.
- During initialization of the vault, a master key is generated and split into
  5 shards
- 3 shards are required to unseal.
  - see [Sharmir's Secret Sharing](https://en.wikipedia.org/wiki/Shamir%27s_Secret_Sharing)
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
    - Tokens
    - AppRole\*
    - GitHub
    - LDAP/AD
    - Username/Password


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

## Seal/Unseal

---

# Vault Operations

## Seal/Unseal

---

# Vault Operations

## Seal/Unseal

