# Steps to reproduce https://github.com/spring-cloud/spring-cloud-config/issues/1997

## 1. Start Vault and add secrets
```
docker-compose up -d
docker-compose exec --env VAULT_TOKEN='localDevelopment' vault vault kv put -address='http://localhost:8200' secret/application foo=bar
docker-compose exec --env VAULT_TOKEN='localDevelopment' vault vault kv put -address='http://localhost:8200' secret/application,profile foo=baz
```


## 2. Start Server
```
cd server
mvn spring-boot:run
```

In another shell:
```
curl -s http://localhost:8080/appl/profile | jq .
```

Result is:
```

{
  "name": "appl",
  "profiles": [
    "profile"
  ],
  "label": null,
  "version": null,
  "state": null,
  "propertySources": [
    {
      "name": "vault:application,profile",
      "source": {
        "foo": "baz"
      }
    },
    {
      "name": "vault:application",
      "source": {
        "foo": "bar"
      }
    },
    {
      "name": "class path resource [configs/application-profile.yml]",
      "source": {
        "source": "testapplication-env_dev"
      }
    },
    {
      "name": "class path resource [configs/application.yml]",
      "source": {
        "source": "testapplication"
      }
    }
  ]
}
```

## 3. Start client
```
cd client
mvn spring-boot:run
```

In another shell:
```
curl -s http://localhost:8081/actuator/health | jq .
```

Result is:
```
{
  "status": "UP",
  "components": {
    "clientConfigServer": {
      "status": "UP",
      "details": {
        "propertySources": [
          "configserver:class path resource [configs/application-profile.yml]",
          "configserver:vault:application,profile",
          "configserver:vault:application",
          "configserver:class path resource [configs/application.yml]",
          "configClient"
        ]
      }
    },
    "diskSpace": {
      "status": "UP",
      "details": {
        "total": 269490393088,
        "free": 206017982464,
        "threshold": 10485760,
        "exists": true
      }
    },
    "ping": {
      "status": "UP"
    },
    "refreshScope": {
      "status": "UP"
    }
  }
}
```
