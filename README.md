https://doc.traefik.io/traefik-mesh/

```
helm repo add traefik https://traefik.github.io/charts
helm repo update
helm install traefik-mesh traefik/traefik-mesh
```

```
helm install traefik-mesh traefik/traefik-mesh --set acl=true
```

Test app

```
kubectl create namespace test
kubectl apply -f server.yaml
kubectl apply -f client.yaml
```

k exec -ti -n test client-5fb5b75-r8hnp -- sh
/ # curl server.test.svc.cluster.local
Hostname: server-79b55c4c65-rmtzt
IP: 127.0.0.1
IP: 10.244.0.78
RemoteAddr: 10.244.0.79:50346
GET / HTTP/1.1
Host: server.test.svc.cluster.local
User-Agent: curl/7.64.0
Accept: */*


/ # curl server.test.traefik.mesh
Hostname: server-79b55c4c65-snzl5
IP: 127.0.0.1
IP: 10.244.0.77
RemoteAddr: 10.244.0.71:53498
GET / HTTP/1.1
Host: server.test.traefik.mesh
User-Agent: curl/7.64.0
Accept: */*
Accept-Encoding: gzip
Uber-Trace-Id: 3af73ad6e5517a53:085e095be8269280:3af73ad6e5517a53:1
X-Forwarded-For: 10.244.0.79
X-Forwarded-Host: server.test.traefik.mesh
X-Forwarded-Port: 80
X-Forwarded-Proto: http
X-Forwarded-Server: traefik-mesh-proxy-hhz8q
X-Real-Ip: 10.244.0.79


```
k apply -f traffictarget.yaml
```

```
$ :) k exec -ti -n test client-6b7cf74cc5-2hfs9 -- sh
/ # curl  server.test.traefik.mesh/aaa
Hostname: server-6cff7c899c-g8rhn
IP: 127.0.0.1
IP: 10.244.0.90
RemoteAddr: 10.244.0.82:40190
GET /aaa HTTP/1.1
Host: server.test.traefik.mesh
User-Agent: curl/7.64.0
Accept: */*
Accept-Encoding: gzip
Uber-Trace-Id: 561cc1f68a2c6909:0744f0a98f6666d8:73c24e3fcdffcada:1
X-Forwarded-For: 10.244.0.88
X-Forwarded-Host: server.test.traefik.mesh
X-Forwarded-Port: 80
X-Forwarded-Proto: http
X-Forwarded-Server: traefik-mesh-proxy-k5b5k
X-Real-Ip: 10.244.0.88

/ # curl -XPOST server.test.traefik.mesh/aaa
Hostname: server-6cff7c899c-mrfkw
IP: 127.0.0.1
IP: 10.244.0.89
RemoteAddr: 10.244.0.82:47942
POST /aaa HTTP/1.1
Host: server.test.traefik.mesh
User-Agent: curl/7.64.0
Content-Length: 0
Accept: */*
Accept-Encoding: gzip
Uber-Trace-Id: 731d6bb9fe017bbf:18a86f9b8d7d5d3d:06f1a8a99716e1cf:1
X-Forwarded-For: 10.244.0.88
X-Forwarded-Host: server.test.traefik.mesh
X-Forwarded-Port: 80
X-Forwarded-Proto: http
X-Forwarded-Server: traefik-mesh-proxy-k5b5k
X-Real-Ip: 10.244.0.88

/ # curl -I server.test.traefik.mesh/aaa
HTTP/1.1 403 Forbidden
Date: Sat, 21 Sep 2024 05:35:38 GMT
Content-Length: 9
Content-Type: text/plain; charset=utf-8

/ # curl server.test.traefik.mesh/api
{"hostname":"server-6cff7c899c-g8rhn","ip":["127.0.0.1","10.244.0.90"],"headers":{"Accept":["*/*"],"Accept-Encoding":["gzip"],"Uber-Trace-Id":["591e0150374c9d92:3e5993cbd4ae5b4d:2f4df7ba2e09fab8:1"],"User-Agent":["curl/7.64.0"],"X-Forwarded-For":["10.244.0.88"],"X-Forwarded-Host":["server.test.traefik.mesh"],"X-Forwarded-Port":["80"],"X-Forwarded-Proto":["http"],"X-Forwarded-Server":["traefik-mesh-proxy-k5b5k"],"X-Real-Ip":["10.244.0.88"]},"url":"/api","host":"server.test.traefik.mesh","method":"GET"}
/ # curl -XPOST server.test.traefik.mesh/api
Forbidden/ #
/ # curl -XPOST server.test.traefik.mesh/bbb
```


```
k apply -f server.yaml
k apply -f trafficsplit.yaml
```

```
$ :) k exec -ti -n test client-6b7cf74cc5-2hfs9 -- sh
/ # for i in $(seq 1 30); do curl server.test.traefik.mesh/aaa; done
aaaaaaaaaaaaaa
aaaaaaaaaaaaaa
bbbbbbbbbbbbbbbb
aaaaaaaaaaaaaa
aaaaaaaaaaaaaa
aaaaaaaaaaaaaa
aaaaaaaaaaaaaa
bbbbbbbbbbbbbbbb
aaaaaaaaaaaaaa
aaaaaaaaaaaaaa
aaaaaaaaaaaaaa
aaaaaaaaaaaaaa
bbbbbbbbbbbbbbbb
aaaaaaaaaaaaaa
aaaaaaaaaaaaaa
aaaaaaaaaaaaaa
aaaaaaaaaaaaaa
bbbbbbbbbbbbbbbb
aaaaaaaaaaaaaa
aaaaaaaaaaaaaa
aaaaaaaaaaaaaa
aaaaaaaaaaaaaa
bbbbbbbbbbbbbbbb
aaaaaaaaaaaaaa
aaaaaaaaaaaaaa
aaaaaaaaaaaaaa
aaaaaaaaaaaaaa
bbbbbbbbbbbbbbbb
aaaaaaaaaaaaaa
aaaaaaaaaaaaaa
/ #
```
