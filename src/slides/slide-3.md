# Other Secret Backends

## AWS

> The AWS secret backend for Vault generates AWS access credentials dynamically
> based on IAM policies. This makes IAM much easier to use: credentials could
> be generated on the fly, and are automatically revoked when the Vault lease
> is expired.

 - The endpoint is configured with the access and secret keys used to manage
   IAM credentials as well as the AWS region
 - A role is created with an IAM policy to be used when accessed
 - When the endpoint is read, an access and secret key for AWS is created.

---

# Other Secret Backends

## AWS

```
mrg@talos$ curl -X GET \
> -H "X-Vault-Token:$VAULT_TOKEN" \
> ${VAULT_ADDR}/v1/aws/creds/ec2readonly |prettyjson 
{
    "auth": null,
    "data": {
        "access_key": "AKIAIPWHLD6HD4AC3UYA",
        "secret_key": "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",
        "security_token": null
    },
    "lease_duration": 3600,
    "lease_id": "aws/creds/ec2readonly/b149fdb3-43b3-6d17-d56f-63593c15bd68",
    "renewable": true,
    "request_id": "9f8f00ed-c63c-0641-2592-3841fb3c595f",
    "warnings": null,
    "wrap_info": null
}
```

---

 # Other Secret Backends

 ## PKI

 The PKI backend allows us to generate certificates based on roles

  - Configure the backend with a signing CA
  - Configure a role based on the CA certificate
  - Write/Post to the role endpoint and a certificate and key are returned


