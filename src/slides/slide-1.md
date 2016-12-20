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

# Introducing Svalbard

> _Svalbard_ is the name I'd chosen for this implementation of Vault

[<img src="https://upload.wikimedia.org/wikipedia/commons/6/6b/Svalbard_seed_vault_IMG_8894.JPG" style="width: 50%; height: 50%"/>](https://www.regjeringen.no/en/topics/food-fisheries-and-agriculture/landbruk/svalbard-global-seed-vault/id462220/)

---

# Svalbard Components

## Consul

[consul](https://www.consul.io/) is a multi-datacenter capbable system providing service discovery, failure detection, and key/value storage.  Vault uses many of Consul's features to provide a highly-available service

I have three Consul servers (really the minimum number required).

---

# Svalbard Components

## A CA

Consul relies heavily on certificates to provide secure communication between nodes.  The Hutch wildcard doesn't appear to work- hence we need self-signed certificates.


---

# Svalbard Components

## Vault

Right now just one vault server- a production roll-out will have more, likely spanning data centers.

---

# Vault Operations

## Authentication

 - Tokens
 - `AppRole`
 - GitHub
 - LDAP/AD
 - Username/Password

---

# Vault Operations

## Seal/Unseal

- Secrets are encrypted- unsealing makes secrets available to the Vault server.
- During initialization of the vault, a master key is generated and split into
  5 shards
- 3 shards are required to unseal.
  - see [Sharmir's Secret Sharing](https://en.wikipedia.org/wiki/Shamir%27s_Secret_Sharing)

???

```
   bin/vault seal
   bin/vault unseal
```

---

# Vault Operations

## Seal/Unseal

---

# Vault Operations

## Seal/Unseal

---

# Vault Operations

## Seal/Unseal

---

# Vault Operations

## Seal/Unseal

