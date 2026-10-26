pierre@pierre:~$ curl D=http://127.0.0.1:2375
curl: (3) URL rejected: Port number was not a decimal number between 0 and 65535
pierre@pierre:~$ D=http://127.0.0.1:2375        # aucune sortie, c'est normal
echo $D                        # vérifie : doit afficher http://127.0.0.1:2375
curl -s $D/version | jq
http://127.0.0.1:2375
{
  "Platform": {
    "Name": "Docker Engine - Community"
  },
  "Version": "29.9.2",
  "ApiVersion": "1.56",
  "MinAPIVersion": "1.40",
  "Os": "linux",
  "Arch": "amd64",
  "Components": [
    {
      "Name": "Engine",
      "Version": "29.9.2",
      "Details": {
        "ApiVersion": "1.56",
        "Arch": "amd64",
        "BuildTime": "2026-10-20T18:45:13.000000000+00:00",
        "Experimental": "false",
        "GitCommit": "0367cbe",
        "GoVersion": "go1.26.10",
        "KernelVersion": "6.12.96+deb13-amd64",
        "MinAPIVersion": "1.40",
        "Module": "github.com/moby/moby/v2",
        "ModuleVersion": "v2.0.0+unknown",
        "Os": "linux"
      }
    },
    {
      "Name": "containerd",
      "Version": "v2.4.2",
      "Details": {
        "GitCommit": "b7a25261e029d68978bbf290df0a62fc819b3c51"
      }
    },
    {
      "Name": "runc",
      "Version": "1.5.2",
      "Details": {
        "GitCommit": "v1.5.2-0-g29dd3dc2"
      }
    },
    {
      "Name": "docker-init",
      "Version": "0.19.0",
      "Details": {
        "GitCommit": "de40ad0"
      }
    }
  ],
  "GitCommit": "0367cbe",
  "GoVersion": "go1.26.10",
  "KernelVersion": "6.12.96+deb13-amd64",
  "BuildTime": "2026-10-20T18:45:13.000000000+00:00"
}
pierre@pierre:~$ cd ~/adminapi
pierre@pierre:~/adminapi$ D=http://127.0.0.1:2375
pierre@pierre:~/adminapi$ curl -s $D/version | jq
{
  "Platform": {
    "Name": "Docker Engine - Community"
  },
  "Version": "29.9.2",
  "ApiVersion": "1.56",
  "MinAPIVersion": "1.40",
  "Os": "linux",
  "Arch": "amd64",
  "Components": [
    {
      "Name": "Engine",
      "Version": "29.9.2",
      "Details": {
        "ApiVersion": "1.56",
        "Arch": "amd64",
        "BuildTime": "2026-10-20T18:45:13.000000000+00:00",
        "Experimental": "false",
        "GitCommit": "0367cbe",
        "GoVersion": "go1.26.10",
        "KernelVersion": "6.12.96+deb13-amd64",
        "MinAPIVersion": "1.40",
        "Module": "github.com/moby/moby/v2",
        "ModuleVersion": "v2.0.0+unknown",
        "Os": "linux"
      }
    },
    {
      "Name": "containerd",
      "Version": "v2.4.2",
      "Details": {
        "GitCommit": "b7a25261e029d68978bbf290df0a62fc819b3c51"
      }
    },
    {
      "Name": "runc",
      "Version": "1.5.2",
      "Details": {
        "GitCommit": "v1.5.2-0-g29dd3dc2"
      }
    },
    {
      "Name": "docker-init",
      "Version": "0.19.0",
      "Details": {
        "GitCommit": "de40ad0"
      }
    }
  ],
  "GitCommit": "0367cbe",
  "GoVersion": "go1.26.10",
  "KernelVersion": "6.12.96+deb13-amd64",
  "BuildTime": "2026-10-20T18:45:13.000000000+00:00"
}
pierre@pierre:~/adminapi$ curl -s "$D/containers/json?all=true" | jq '.[] | {Names, Image, State}'
{
  "Names": [
    "/intrus"
  ],
  "Image": "nginx:alpine",
  "State": "running"
}
{
  "Names": [
    "/web1"
  ],
  "Image": "nginx:alpine",
  "State": "running"
}
{
  "Names": [
    "/apirest-netbox-1"
  ],
  "Image": "netboxcommunity/netbox:v4.7-5.1.1",
  "State": "running"
}
{
  "Names": [
    "/apirest-netbox-postgres-1"
  ],
  "Image": "postgres:17-alpine",
  "State": "running"
}
{
  "Names": [
    "/apirest-docker-api-1"
  ],
  "Image": "tecnativa/docker-socket-proxy:v0.4.2",
  "State": "running"
}
{
  "Names": [
    "/apirest-netbox-redis-1"
  ],
  "Image": "valkey/valkey:8-alpine",
  "State": "running"
}
pierre@pierre:~/adminapi$ curl -s --get $D/containers/json --data-urlencode 'all=true' \
     --data-urlencode 'filters={"label":["adminapi.managed=true"]}' | jq '.[] | {Names, Image, State}'
{
  "Names": [
    "/web1"
  ],
  "Image": "nginx:alpine",
  "State": "running"
}
pierre@pierre:~/adminapi$ curl -s $D/containers/web1/json | jq '{Name, State: .State.Status, Labels: .Config.Labels}'
{
  "Name": "/web1",
  "State": "running",
  "Labels": {
    "adminapi.managed": "true",
    "maintainer": "NGINX Docker Maintainers <docker-maint@nginx.com>"
  }
}
pierre@pierre:~/adminapi$ echo "1er stop :";        curl -s -o /dev/null -w "%{http_code}\n" -X POST $D/containers/web1/stop
1er stop :
204
pierre@pierre:~/adminapi$ echo "2e stop :";         curl -s -o /dev/null -w "%{http_code}\n" -X POST $D/containers/web1/stop
2e stop :
304
pierre@pierre:~/adminapi$ echo "inexistant :";      curl -s -o /dev/null -w "%{http_code}\n" $D/containers/nexistepas/json
inexistant :
404
pierre@pierre:~/adminapi$



pierre@pierre:~/adminapi$ N=http://127.0.0.1:8080/api
curl -s -X POST $N/users/tokens/provision/ \
     -H "Content-Type: application/json" -H "Accept: application/json" \
     -d '{"username": "admin", "password": "admin"}' | jq
{
  "id": 1,
  "url": "http://127.0.0.1:8080/api/users/tokens/1/",
  "display_url": "http://127.0.0.1:8080/users/tokens/1/",
  "display": "vbZpKB1jJdVT",
  "version": 2,
  "user": {
    "id": 1,
    "url": "http://127.0.0.1:8080/api/users/users/1/",
    "display": "admin",
    "username": "admin"
  },
  "key": "vbZpKB1jJdVT",
  "created": "2026-10-26T11:36:25.220016Z",
  "expires": null,
  "last_used": null,
  "enabled": true,
  "write_enabled": true,
  "description": "",
  "allowed_ips": [],
  "token": ""
}
pierre@pierre:~/adminapi$





set -a; source .env; set +a
A="Authorization: Bearer $NETBOX_TOKEN"
N=http://127.0.0.1:8080/api

TYPE_ID=$(curl -s -X POST $N/virtualization/cluster-types/ -H "$A" \
     -H "Content-Type: application/json" -H "Accept: application/json" \
     -d '{"name": "Docker", "slug": "docker"}' | jq '.id')
echo "id du type : $TYPE_ID"

curl -s -X POST $N/virtualization/clusters/ -H "$A" \
     -H "Content-Type: application/json" -H "Accept: application/json" \
     -d "{\"name\": \"docker-local\", \"type\": $TYPE_ID}" | jq '{id, name}'
