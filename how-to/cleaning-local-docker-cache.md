## Cleaning local docker cache

When using docker to frequently build containers and images the local docker caches tend to fill up. Eventually they will take up an undesirable amount of disk space. Below are a few handy commands to help keep control of your local docker caches.


#### Remove containers with specific status

```bash
docker rm -v $(docker ps -qa --no-trunc --filter "status=exited" --filter "status=dead" --filter "status=created")
```

#### Remove "dangling" docker images

```bash
docker rmi $(docker images --filter "dangling=true" -q --no-trunc)
```

#### Remove previously built MUSIT service images

```bash
docker rmi $(docker images -a | grep "^service_*" | awk '{print $3}') --force
```

#### Remove unnamed and untagged images

```bash
docker rmi $(docker images -a --no-trunc | grep "^<none>" | awk '{print $3}')
```