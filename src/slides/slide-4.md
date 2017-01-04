# Auditing

 * Vault maintains a record of every interaction
 * Store in a file or syslog (then to Splunk if desired)
 * Multiple audit backends are allowed
 * Secrets are hashed in the audit log

---

# Vault Libraries

 * _Go_ and _Ruby_ libraries are officially supported
 * _Python_, _C#_, _Node.js_ are just three of the community supported
 * [Full List](https://www.vaultproject.io/docs/http/libraries.html)

---

# Python's HVAC Library

```
import hvac

# Using plaintext
client = hvac.Client()
client = hvac.Client(url='http://localhost:8200')
client = hvac.Client(
           url='http://localhost:8200',
           token=os.environ['VAULT_TOKEN']
         )
client.write('secret/foo', baz='bar', lease='1h')
print(client.read('secret/foo'))
client.delete('secret/foo')
```

---

# Wrap Up

 * Secrets are *hard*
 * Managing secrets is not encrypting- managing access and authentication is
 * Moving to cloud computing only exacerbates complexities
 * What seems like overkill today may be tomorrows base functionality

## Value of Vault

 * Strong multipart encryption of secrets
   * No one person with complete access (if desired)
 * Well-thought-out authentication schemes
 * Ephemeral secret generation and management

???

 * The ephemeral secrets are what I'd consider the "next generation" of secret
   management.  Services are only given access when needed, access removed when
   complete.  No reuse

---

# Versus Chef Vault

  * Chef Vault is only a key/value store
  * Administrative access controls are likely sufficient
  * Client certificate is used for authentication
  * Integrated nicely with configuration management
  * Limited management of secrets, simply access control
  * No expiry, no advanced secrets (e.g. Postgres credential generation)

---

# Versus AWS Key Management Service

  * KMS only stores cryptographic keys
  * Not designed for storing the secret itself (but could be used for managing
    access)

---

# Versus Keywhiz (Square)

  * Also designed for comprehensive secret management
  * Cookie or certificate based access control
  * API, CLI, or management web interface interactions
  * Can mount secrets via FUSE using Unix ACL for access control
  * Much more in line with Chef Vault in execution
  * No apparent mechanism for ephemeral secrets or managing service secrets


