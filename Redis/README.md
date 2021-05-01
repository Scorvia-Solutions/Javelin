# Kubernetes Redis Cluster Setup 
Authored by WarpWing
## Description 

This is a short tutorial on how to deploy a Kubernetes Redis Cluster (Adapted heavily from a Rancher Blog.)

# Requirements
- 42 GBs of spare spaces for Persistence Volumes 
- A Kubernetes Cluster (Nodes count doesn't matter)
- Basic commandline experience

## Infrastructure Model 

![](https://rancher.com/img/blog/2018/deploying-redis-cluster/01-rancher-redis-cluster-architecture_hu26fb1ab7510f6a361f82914dbd2d473e_116029_1000x0_resize_box_2.png)

# Context

The diagram that you see above you is a model of the Redis Cluster Communication Infrastructure. In the diagram, scaling is done by partitioning it. The cluster works off of 3 Master-Worker models. If a Master fails or becomes unreachable, the Worker instance gets promoted to Master while a new Worker instance is generated. 

Note that the Master node is always the front facing node while the Worker node is the reserved node on standby. Note that no matter if the instance is a Master or a Worker, they all share the data equally which if a Master fails, the Worker node will instantly take over with the same state as the just failed Master. Keeping technical details short, Communication inside the Cluster is made via a internal bus network known as "gossip" which helps to distribute information about the cluster or discover new nodes.

# Instructions 

Please follow these instructions from top to bottom. I am not responsible if you mess up your Cluster so please proceed with caution 

- Apply the Stateful Set and ConfigMap YAML file by doing the following:
```shell
kubectl apply -f redis-sts-cfg.yml
``` 
- Apply the Redis Load Balancer file by doing the following: 
```shell 
kubectl apply -f redis-svc.yaml
``` 
- Deploy the actual Cluster (The first two steps was to provide the initial infrastructure and configuration) by doing the following: 
```shell 
kubectl exec -it redis-cluster-0 -- redis-cli --cluster create --cluster-replicas 1 $(kubectl get pods -l app=redis-cluster -o jsonpath='{range.items[*]}{.status.podIP}:6379 ')
```
- Verify that the cluster deployed successfully(Give the Kubernetes Cluster time to spin up the instances. 2-5 minutes max) by doing the following 
```shell
for x in $(seq 0 5); do echo "redis-cluster-$x"; kubectl exec redis-cluster-$x -- redis-cli role; echo; done
```
Note: This command should return six nodes with the first 3 being the master nodes and the others being the worker/slave nodes. It should be fine if it's interchangable but as long as there is three master nodes and three worker/slave nodes, you should be ok :)

## Conclusion

Wow you actually made it through! If you're reading this, you should have successfully created a Redis Cluster on Kubernetes! Woohoo! If you have any questions, please feel free to hit me up on Discord at WarpWing#3866 or just open a issue on Github. If you have any problems with this writeup please let me know! It's my first time writing and most of the commands are based off of a Rancher Blog so I just rewrote it so that it would be easily digestable to the readers who visit this repository. Anyways I wish those who completed good luck and high uptime in the future!

