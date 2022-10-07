# My homepage

## Setup

This app is separated into two submodules both on GitLab and each has its pipelines. This pipeline only refreshes services and the main proxy. 
All docker images are built for arm64 architecture.



### Submodules
```
$ git submodule init
$ git submodule update
```


### Project setup
Each project has its .env file, which is structured like .env.dist files.

### CI/CD setup
If using a specific runner, it needs to contain the following configuration:
```
environment = ["DOCKER_HOST=tcp://localhost:2375", "DOCKER_TLS_CERTDIR="]
privileged = true

```
Pipelines have subsets of variables, necessary to run them. The Main Site repo has the following: 
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

## Run the project
This module has only images so no need to build to run the main proxy use following:
```
$ docker-compose up
```


