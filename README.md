# Rocket Tutorial

```
package main

import (
    "log"
    "net/http"
)

func main() {
    http.HandleFunc("/", func(w http.ResponseWriter, r *http.Request) {
        log.Printf("request from %v\n", r.RemoteAddr)
        w.Write([]byte("hello\n"))
    })
    log.Fatal(http.ListenAndServe(":5000", nil))
}
```

```
CGO_ENABLED=0 GOOS=linux go build -a -tags netgo -ldflags '-w' .
```

```
{
    "acVersion": "1.0.0",
    "acKind": "AppManifest",
    "name": "coreos.com/hello-1.0.0",
    "os": "linux",
    "arch": "amd64",
    "exec": [
        "/bin/hello"
    ],
    "ports": [
        {
            "name": "www",
            "protocol": "tcp",
            "port": 5000
        }
    ],
    "annotations": {
        "authors": "Kelsey Hightower <kelsey.hightower@gmail.com>"
    }
}
```

```
actool validate manifest.json
```

```
mkdir rootfs
mkdir rootfs/bin
```

```
cp hello rootfs/bin/
```

```
actool build --app-manifest manifest.json rootfs hello.aci
```

```
actool validate hello.aci
```

### Launch a local application image

```
rkt run hello.aci
```

### Testing with curl

```
curl 127.0.0.1:5000
```

## Hosting App Container Images

### Install a webserver

```
brew install nginx
nginx
```

### Copy the binary

```
scp core@192.168.2.136:~/hello/hello.aci .
mv hello.aci /usr/local/var/www/
```

### Run it.

```
rkt run http://192.168.2.1:8080/hello.aci
```

## Launch a Docker Container with Rocket

### Download and export a Docker Container

```
docker pull coreos/etcd
```

```
docker run --name=etcd coreos/etcd
```

### Export the Docker Container

```
mkdir rootfs
```

```
docker export etcd | sudo tar -x -C rootfs -f -
```

### Create the app manifest

```
{
    "acVersion": "1.0.0",
    "acKind": "AppManifest",
    "name": "coreos.com/etcd",
    "os": "linux",
    "arch": "amd64",
    "exec": [
        "/etcd -name node0"
    ],
    "ports": [
        {
            "name": "etcdclient",
            "protocol": "tcp",
            "port": 4001
        },
        {
            "name": "etcdraft",
            "protocol": "tcp",
            "port": 7001
        }

    ],
    "annotations": {
        "authors": "Kelsey Hightower <kelsey.hightower@gmail.com>"
    }
}
```

### Build the container

```
actool build --app-manifest manifest.json rootfs etcd.aci
```

### Launch the container with Rocket

```
sudo rkt run etcd.aci
```
