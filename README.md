# UniFi Network

## MongoDB Setup

You need to use an external MongoDB instance. In my case, I've simply set up a
new server in my Kubes cluster, inspired but not used unmodified: https://github.com/techiescamp/kubernetes-mongodb

Create the required `unifi` user. Compared to the unifi docker documentation, we
need to two roles for the `unifi` user on the `unifi` db, otherwise the service
would not start up correctly. (Rename db and user as you fancy, and don't forget
to set a proper password.)

```
db.getSiblingDB('unifi').createUser({user: "unifi", pwd: "pass", roles: [{role: "dbOwner", db: "unifi"}, {role: "dbOwner", db: "unifi_stat"}]})
db.getSiblingDB('unifi_stat').createUser({user: "unifi", pwd: "pass", roles: [{role: "dbOwner", db: "unifi_stat"}]})
```

## Install

I did not want to create a Helm chart to make it versatile, so you need to
change the following.

Open and edit the `deployment.yaml`. Most of it might be ok to proceed with,
however, check the service IP etc., `spec.type` and `spec.loadBalancerIP` in the
Kubernetes Service section.

Create the required Secret:

```
# Replace with your values incl. the namespace if you changed it.
kubectl create secret generic unifi-secrets -n unifi \
  --from-literal=MONGO_HOST={db-host} \
  --from-literal=MONGO_PORT=27017 \
  --from-literal=MONGO_DBNAME={db-name} \
  --from-literal=MONGO_USER={db-user} \
  --from-literal=MONGO_PASS={db-password} \
  --from-literal=MONGO_TLS=false
```

Run `kubectl apply -f deployment.yaml`.

If you want to expose it on the internet using Envoy Gateway, run `kubectl apply
-f httproute.yaml`. Since we need to skip validation of the backend tls cert,
there's a bit of extra setup with the `Backend` resource. Also, remember to
enable the [`Backend`](https://gateway.envoyproxy.io/docs/tasks/traffic/backend/#enable-backend)
API.

## Connect AP

If an AP does not show up automatically, you can SSH into it while still in
factory settings.

```
ssh ubnt@{the-AP's-ip}
```

Password is `ubnt`.

Then run `set-inform http://{the-server's-ip}:8080/inform` and it should show up
in the server's web interface.
