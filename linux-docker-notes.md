# Docker host.docker.internal
Works only for Windows and Mac as granted, for Linux we have to provide:
- "host.docker.internal:host-gateway"
In docker-compose.yml we have to put it for each service:
```yml
...
services:
    some_service_name:
        extra_hosts:
        - "host.docker.internal:host-gateway"
...
```