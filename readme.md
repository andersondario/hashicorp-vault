# Hashi Vault

### Descrição
Esse projeto contém os arquivos necessários para a construção de um servidor Hashi Vault.

### Configuração
Os detalhes de configuração do servidor devem ser definidos no arquivo _vault/config/vault-config.json_

### Destravando o servidor
Sempre que o servidor for iniciado, vai ser necessário destravar ele com um conjunto de chaves. No primeiro login é necessário definir em quantas porções a chave de destravamento será quebrada e quantas porções dele serão exigidas para o destravamento. Após destravar, basta usar o token master para logar na aplicação.

### API
O servidor possui uma API no qual é possível interagir. Abaixo é demonstrado alguns exemplos de uso.
1. Caso esteja utilizando user/password, o login é feito da seguinte forma:
```shell
curl -X POST \ 
    -H "Content-Type: application/json" \ 
    -d '{ "password": $USER_PASS }' $VAULT_URL/v1/auth/userpass/login/$USER_NAME
```

2. Caso queira fornecer credenciais para Apps, é indicado o uso da autenticação [Approle](https://learn.hashicorp.com/tutorials/vault/approle). Para isso é necessário:<br/>
a) Criação de uma role
```shell
curl -X POST \
    -H 'Authorization: Bearer $VAULT_TOKEN' \
    -H "Content-Type: application/json" \ 
    -d '{ "token_ttl": "10m", "token_policies": ["$POLICY_NAME"] }' $VAULT_URL/v1/auth/approle/role/$ROLE_NAME 
```
b) Ler o role_id da role criada acima
```shell
curl -s \ 
    -H 'Authorization: Bearer $VAULT_TOKEN' $VAULT_URL/v1/auth/approle/role/$ROLE_NAME/role-id
```
c) Criar um secret_id para a role acima
```shell
curl -X POST \ 
    -H 'Authorization: Bearer $VAULT_TOKEN' $VAULT_URL/v1/auth/approle/role/$ROLE_NAME/secret-id
```
d) Realizar login
```shell
curl -X POST \
    -H "Content-Type: application/json" \ 
    -d '{ "role_id": $ROLE_ID, "secret_id": $SECRET_ID }' $VAULT_URL/v1/auth/approle/login
```

3. Obter chaves armazenadas em um secret de tipo kv (key-value) versão 1.
```shell
curl -s -H 'Authorization: Bearer $VAULT_TOKEN' $VAULT_URL/v1/$SECRET_BUCKET_NAME/$SECRET_NAME 
```

A requisição acima retorna todos as keys e values, junto com alguns metadados. Para pegar um value específico, precisamos utilizar um processador de JSON (jq por exemplo), conforme será mostrado a seguir

```shell
curl -s -H 'Authorization: Bearer $VAULT_TOKEN' $VAULT_URL/v1/$SECRET_BUCKET_NAME/$SECRET_NAME | jq -r ".data.$KEY_NAME"
```

### Links úteis
- [Documentação oficial](https://www.vaultproject.io/docs)
- [Documentação oficial - API] (https://www.vaultproject.io/api-docs)
- [Recomendações para subida em produção com Docker](https://learn.hashicorp.com/tutorials/vault/production-hardening?in=vault/day-one-raft)
- [Recomendações para subida em produção com Openshift](https://www.vaultproject.io/docs/platform/k8s/helm/openshift)