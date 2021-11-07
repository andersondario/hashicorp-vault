# Hashi Vault

### Description
This project contains the necessary files to run a single Hashicorp Vault server for non-production purpouses.

### Configuration
The configuration details of the server are defined on the file _vault/config/vault-config.json_

### Unlock the server
Always when the server goes up, it will be necessary unlock it with a set of keys. In the first login is need to define how many unlock keys will be generated for unlock the server. After unlock the server, login with the master token.

### API
The server has an API which is possible to interate. Look on the examples below:
1. If you're using user/password authentication:
```shell
curl -X POST \ 
    -H "Content-Type: application/json" \ 
    -d '{ "password": $USER_PASS }' $VAULT_URL/v1/auth/userpass/login/$USER_NAME
```

2. If you want to give credentials for Apps, is indicated to use authentication by [Approle](https://learn.hashicorp.com/tutorials/vault/approle). For this, it is necessary:<br/>
a) Create a role
```shell
curl -X POST \
    -H 'Authorization: Bearer $VAULT_TOKEN' \
    -H "Content-Type: application/json" \ 
    -d '{ "token_ttl": "10m", "token_policies": ["$POLICY_NAME"] }' $VAULT_URL/v1/auth/approle/role/$ROLE_NAME 
```
b) Read the the role_id 
```shell
curl -s \ 
    -H 'Authorization: Bearer $VAULT_TOKEN' $VAULT_URL/v1/auth/approle/role/$ROLE_NAME/role-id
```
c) Create a secret_id for the role
```shell
curl -X POST \ 
    -H 'Authorization: Bearer $VAULT_TOKEN' $VAULT_URL/v1/auth/approle/role/$ROLE_NAME/secret-id
```
d) Do the login
```shell
curl -X POST \
    -H "Content-Type: application/json" \ 
    -d '{ "role_id": $ROLE_ID, "secret_id": $SECRET_ID }' $VAULT_URL/v1/auth/approle/login
```

3. Get the keys from a Vault of kv type (key-value) version 1.
```shell
curl -s -H 'Authorization: Bearer $VAULT_TOKEN' $VAULT_URL/v1/$SECRET_BUCKET_NAME/$SECRET_NAME 
```

The request above will get all key/values from the secret. You can extract only the necessary for you with the jq, like that:

```shell
curl -s -H 'Authorization: Bearer $VAULT_TOKEN' $VAULT_URL/v1/$SECRET_BUCKET_NAME/$SECRET_NAME | jq -r ".data.$KEY_NAME"
```

### References
- [Documentação oficial](https://www.vaultproject.io/docs)
- [Documentação oficial - API] (https://www.vaultproject.io/api-docs)
- [Recomendações para subida em produção com Docker](https://learn.hashicorp.com/tutorials/vault/production-hardening?in=vault/day-one-raft)
- [Recomendações para subida em produção com Openshift](https://www.vaultproject.io/docs/platform/k8s/helm/openshift)
