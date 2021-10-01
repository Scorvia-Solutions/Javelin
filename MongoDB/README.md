# MongoDB setup for SkyCrypt Javelin

https://hub.kubeapps.com/charts/bitnami/mongodb-sharded
```bash
helm install v1 bitnami/mongodb-sharded --set mongos.replicas=4 --set serviceType=NodePort
```

Just install that lmao. More detailed stuff coming out soon
