# My homepage

## Setup

This app is separated into two submodules both on gitlab and each has its own pipelines. This pipline only refreshes services and the main proxy. 
All docker immmgaes are built for arm64 architecture.



### Submodules
```
$ git submodule init
$ git submodule update
```


### Project setup
Each project has own .env file, which are structured like .env.dist files.

### CI/DI setup
If using specific runner, it needs to contain following configuration:
```
environment = ["DOCKER_HOST=tcp://localhost:2375", "DOCKER_TLS_CERTDIR="]
privileged = true

```
Pipelines have subsets of variables, necessary to run them. Main Site repo has following: 
```
CI_REGISTRY: [registry.gitlab.com]
CI_REGISTRY_PASSWORD: [passwd]
CI_REGISTRY_USER: [user]
ENV_FILE: [.env.dist]
PROD_SERVER_IP: [domain]
SSH_CONFIG: [
  Host [domain]
  ProxyCommand cloudflared access ssh --hostname %h
  ]
SSH_PRIVATE_KEY: [pk]
SSH_ROOT_PRIVATE_KEY: [root enabled pk]
```

## Run
This module has only immages so no need to build to run the main proxy use following:
```
$ docker compose up
```


