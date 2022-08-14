# kong-konga
Configuração de um ambiente com konga e konga

## install kong
```
docker volume create kong-vol
```
```
docker network create kong-net
```
```
docker run -d --name kong-database --network=kong-net -e "POSTGRES_USER=kong" -e "POSTGRES_DB=kong" -e "POSTGRES_PASSWORD=kong" -p 5442:5432 postgres:9.6
```

```
docker run --rm --network=kong-net -e "KONG_DATABASE=postgres" -e "KONG_PG_HOST=kong-database" -e "KONG_PG_PASSWORD=kong" kong:latest kong migrations bootstrap
```

```
docker run -d --name kong --network=kong-net -e "KONG_DATABASE=postgres" -e "KONG_PG_HOST=kong-database" -e "KONG_PG_PASSWORD=kong" -e "KONG_PROXY_ACCESS_LOG=/dev/stdout" -e "KONG_ADMIN_ACCESS_LOG=/dev/stdout" -e "KONG_PROXY_ERROR_LOG=/dev/stderr" -e "KONG_ADMIN_ERROR_LOG=/dev/stderr" -e "KONG_ADMIN_LISTEN=0.0.0.0:8001, 0.0.0.0:8444 ssl" -p 8000:8000 -p 8443:8443 -p 8001:8001 -p 8444:8444 kong:latest
```

## install konga

```
docker volume create konga-postgresql
```
```
docker run -d --name konga-database  --network=kong-net -p 5433:5432 -v konga-postgresql:/var/lib/postgresql/data -e "POSTGRES_USER=konga" -e "POSTGRES_DB=konga" -e "POSTGRES_PASSWORD=konga" postgres:9.6
```
```
docker run --rm  --network=kong-net pantsel/konga:latest -a postgres -c prepare -u postgres://konga:konga@konga-database:5432/konga
```
```
docker run -d -p 1337:1337 --network kong-net -e "DB_ADAPTER=postgres" -e "DB_URI=postgres://konga:konga@konga-database:5432/konga" -e "NODE_ENV=production" -e "DB_PASSWORD=konga" --name konga pantsel/konga
```

## configurar 

acessar: http://localhost:1337/

Criar usuário e senha, 
Usuário: admin
Senha: adminadmin

## configurar kong 

Importar no postman:

### Add Kong Admin API as services

```
curl --location --request POST 'http://localhost:8001/services/' --header 'Content-Type: application/json' --data-raw '{"name":"admin-api","host":"localhost","port": 8001}'
```

Resultado:

```
{
    "host": "localhost",
    "tls_verify_depth": null,
    "tags": null,
    "ca_certificates": null,
    "retries": 5,
    "path": null,
    "created_at": 1660514892,
    "updated_at": 1660514892,
    "client_certificate": null,
    "name": "admin-api",
    "read_timeout": 60000,
    "protocol": "http",
    "port": 8001,
    "enabled": true,
    "write_timeout": 60000,
    "connect_timeout": 60000,
    "id": "864c35c5-362b-4ccf-bbeb-a5c21033e5ec",
    "tls_verify": null
}
```

### Add Admin API route: To register route on Admin API Services we need either service name or service id, you can replace the following command below:

```
curl --location --request POST 'http://localhost:8001/services/admin-api/routes' \
--header 'Content-Type: application/json' \
--data-raw '{
    "paths": ["/admin-api"]
}'
```

result:

```
{
    "methods": null,
    "sources": null,
    "destinations": null,
    "snis": null,
    "headers": null,
    "regex_priority": 0,
    "created_at": 1660515539,
    "updated_at": 1660515539,
    "https_redirect_status_code": 426,
    "request_buffering": true,
    "response_buffering": true,
    "tags": null,
    "hosts": null,
    "service": {
        "id": "864c35c5-362b-4ccf-bbeb-a5c21033e5ec"
    },
    "strip_path": true,
    "preserve_host": false,
    "name": null,
    "protocols": [
        "http",
        "https"
    ],
    "path_handling": "v0",
    "id": "494f8ade-23b2-4dcd-88bb-4cb707847bcc",
    "paths": [
        "/admin-api"
    ]
}
```

## Enable Key Auth Plugin

```
curl -X POST http://localhost:8001/services/admin-api/plugins --data "name=key-auth" 
```

result:

```
{
    "created_at": 1660515836,
    "protocols": [
        "grpc",
        "grpcs",
        "http",
        "https"
    ],
    "enabled": true,
    "name": "key-auth",
    "route": null,
    "tags": null,
    "service": {
        "id": "864c35c5-362b-4ccf-bbeb-a5c21033e5ec"
    },
    "consumer": null,
    "id": "49b38913-51cc-4e7b-ac7e-5e12c4987e34",
    "config": {
        "key_in_header": true,
        "key_in_query": true,
        "key_in_body": false,
        "run_on_preflight": true,
        "anonymous": null,
        "key_names": [
            "apikey"
        ],
        "hide_credentials": false
    }
}
```

## Add Konga as Consumer

```
curl --location --request POST 'http://localhost:8001/consumers/' --form 'username=konga' --form 'custom_id=cebd360d-3de6-4f8f-81b2-31575fe9846a'
```

result:

```
{
    "created_at": 1660516081,
    "username": "konga",
    "custom_id": "cebd360d-3de6-4f8f-81b2-31575fe9846a",
    "id": "52e47ae6-cfc2-4b62-8f37-70b95b71b2e0",
    "tags": null
}
```

## 

```
http://localhost:8001/consumers/52e47ae6-cfc2-4b62-8f37-70b95b71b2e0/key-auth
```

result:

```
{
    "created_at": 1660516155,
    "consumer": {
        "id": "52e47ae6-cfc2-4b62-8f37-70b95b71b2e0"
    },
    "key": "dSDb35QlwUxFggyTeiiFWTedIkAwcsfQ",
    "tags": null,
    "id": "160f2964-af76-4102-9d64-1c42863a729f",
    "ttl": null
}
```

## Configuração da conexão no kong

Tipo: key auth
Name: kong
Loopback API URL: http://kong:8000/admin-api
API KEY: dSDb35QlwUxFggyTeiiFWTedIkAwcsfQ



