# My homepage

## Setup

This app is separated into two submodules both on gitlab and each has its own pipelines. This pipline only refreshes services and the main proxy.

### Submodules
```
$ git submodule init
$ git submodule update
```

This module has only immages so no need to build to run the main proxy use following:
## Run
```
$ docker compose up
```


