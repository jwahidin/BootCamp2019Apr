# Studying MCM components
Login to the environment.
Replace the IP address to the correct version for your environment.
```
[root@icp2m1 ~]# cloudctl login -a https://172.16.254.203:8443 -u admin -n kube-system --skip-ssl-validation

Password>
Authenticating...
OK

Targeted account smallcluster Account (id-smallcluster-account)

Targeted namespace kube-system

Configuring kubectl ...
Property "clusters.smallcluster" unset.
Property "users.smallcluster-user" unset.
Property "contexts.smallcluster-context" unset.
Cluster "smallcluster" set.
User "smallcluster-user" set.
Context "smallcluster-context" created.
Switched to context "smallcluster-context".
OK

Configuring helm: /root/.helm
OK
```

## Hub Controller.
### PODS
Notice the pods that made up the Hub Controller (you can also use the release name)
```
[root@icp3m1 ~]# kubectl get pods | grep ibm-mcm-
mcm-release-ibm-mcm-prod-apiserver-7f878dc8fd-4rp9t               1/1     Running     2          11d
mcm-release-ibm-mcm-prod-controller-5555c8bdbd-96s7j              3/3     Running     9          11d
mcm-release-ibm-mcm-prod-etcd-0                                   1/1     Running     0          11d
mcm-release-ibm-mcm-prod-mcm-search-b56cf495c-247h7               2/2     Running     0          11d
mcm-release-ibm-mcm-prod-mcmui-5c48cbd4fd-7l6v4                   1/1     Running     0          11d
mcm-release-ibm-mcm-prod-mcmuiapi-7ff87cf85c-cjr9j                1/1     Running     0          11d
mcm-release-ibm-mcm-prod-mongodb-c77fff7b-2d9jz                   1/1     Running     0          11d
mcm-release-ibm-mcm-prod-platform-header-bmcmg                    1/1     Running     0          11d
mcm-release-ibm-mcm-prod-prometheus-97c49b554-nm84c               3/3     Running     6          11d
mcm-release-ibm-mcm-prod-webhook-6cb8cf5d7b-r9gm8                 1/1     Running     0          11d
```
Note each of the components
- that looks like the standard kubernetes components (eg. apiserver, controller, etcd),
- that handles the user interface (mcmui, mcmuiapi, mcm-search)
- data (mongodb)
- monitoring (prometheus)
- webhook

## Managed Clusters
### Managed Clusters at Hub Controller
Notice the pods that made up the Managed Clusters that run at the Hub Controller:
(substitute the klusterletrelease with your release name, or use `kubctl get pods | grep ibm-mcmk` )
```
[root@icp3m1 ~]# kubectl get pods -l release=mcm-klusterlet-rel
NAME                                                              READY   STATUS    RESTARTS   AGE
mcm-klusterlet-rel-ibm-mcmk-prod-klusterlet-6cf75667c4-rh2vt      3/3     Running   0          11d
mcm-klusterlet-rel-ibm-mcmk-prod-weave-scope-app-6d55977c4gzw6q   2/2     Running   0          11d
mcm-klusterlet-rel-ibm-mcmk-prod-weave-scope-gjfgb                1/1     Running   0          11d
mcm-klusterlet-rel-ibm-mcmk-prod-weave-scope-sjsnx                1/1     Running   0          11d
```
Note there are 3 containers running in the klusterlet pod.

Run the describe
```
[root@icp3m1 ~]# kubectl describe pod mcm-klusterlet-rel-ibm-mcmk-prod-klusterlet-6cf75667c4-rh2vt
Name:               mcm-klusterlet-rel-ibm-mcmk-prod-klusterlet-6cf75667c4-rh2vt
Namespace:          kube-system
Priority:           0
PriorityClassName:  <none>
Node:               172.16.254.89/172.16.254.89
Start Time:         Fri, 22 Mar 2019 23:50:24 +1100
Labels:             app=ibm-mcmk-prod
                    chart=ibm-mcmk-prod-3.1.2
                    component=klusterlet
                    heritage=Tiller
                    pod-template-hash=6cf75667c4
                    release=mcm-klusterlet-rel
Annotations:        kubernetes.io/psp: ibm-privileged-psp
                    productID: 354b8990aab44c9988a0edfda101b128
                    productName: IBM Multi-cloud Manager - Klusterlet
                    productVersion: 3.1.2
Status:             Running
IP:                 10.1.42.214
Controlled By:      ReplicaSet/mcm-klusterlet-rel-ibm-mcmk-prod-klusterlet-6cf75667c4
Containers:
  work-manager:
    Container ID:  docker://dffedc4da71227e50130460505bad953ce8e28a0a44ebbcd5daefa6c29d8c9b1
    Image:         cemcluster.icp:8500/kube-system/mcm-klusterlet:3.1.2
    Image ID:      docker-pullable://cemcluster.icp:8500/kube-system/mcm-klusterlet@sha256:933daaadb55fa0a2c90e0e0dd9a53e0ff9d3bf7e5be6ddec6f3d84270f1c8a9f
    Port:          <none>
    Host Port:     <none>
    Args:
      /klusterlet
      --bootstrap-controller-config-file=/var/run/bootstrap-klusterlet/config
      --client-cert-secret=kube-system/mcm-klusterlet-rel-ibm-mcmk-prod-klusterlet-cert
      --controller-config-file=/var/run/klusterlet/config
      --cluster-name=mcm-cem-cluster
      --cluster-namespace=mcm-cem-cluster-ns
      --cluster-labels=cloud=IBM,vendor=ICP,environment=QA,region=AP,datacenter=Melbourne,owner=csmo
      --tiller-cert=/etc/helm/tls.crt
      --tiller-key=/etc/helm/tls.key
      --helm-release-prefix=md
      --weave-host=https://mcm-klusterlet-rel-ibm-mcmk-prod-weave-scope.kube-system:443
      --weave-interval=15
      --weave-cert=/opt/ibm/router/client-certs/tls.crt
      --weave-key=/opt/ibm/router/client-certs/tls.key
      --klusterlet-ingress=kube-system/mcm-klusterlet-rel-ibm-mcmk-prod-klusterlet
      --klusterlet-address=mcm-cem-cluster.klusterlet.mcm
    State:          Running
      Started:      Fri, 22 Mar 2019 23:50:29 +1100
    Ready:          True
    Restart Count:  0
    Environment:    <none>
    Mounts:
      /etc/helm from tiller-secret (rw)
      /opt/ibm/router/client-certs from weavescope-ca-secret (rw)
      /var/run/bootstrap-klusterlet from klusterlet-config (rw)
      /var/run/secrets/kubernetes.io/serviceaccount from default-token-rk8bc (ro)
  policy-controller:
    Container ID:  docker://987e0c3107265a297cb481093d8e3f4652ef1b4fbcb6855770cb65ad16b70d62
    Image:         cemcluster.icp:8500/kube-system/mcm-compliance:3.1.2
    Image ID:      docker-pullable://cemcluster.icp:8500/kube-system/mcm-compliance@sha256:f67a1a692e5f5b96b89df0ba7ff557dfefe0ebe1161509ab71fd7d09109a4d65
    Port:          <none>
    Host Port:     <none>
    Command:
      policy-controller
    Args:
      -logtostderr=true
      --watch-ns=mcm-cem-cluster-ns
      --cluster-name=mcm-cem-cluster
      -v=5
      --alert-target=true
    State:          Running
      Started:      Fri, 22 Mar 2019 23:50:29 +1100
    Ready:          True
    Restart Count:  0
    Environment:    <none>
    Mounts:
      /var/run/secrets/kubernetes.io/serviceaccount from default-token-rk8bc (ro)
  compliance-controller:
    Container ID:  docker://d3751ffd8786f70bf021823e59e5e9ce5eb5c2f07d881cbab13ec029ec421303
    Image:         cemcluster.icp:8500/kube-system/mcm-compliance:3.1.2
    Image ID:      docker-pullable://cemcluster.icp:8500/kube-system/mcm-compliance@sha256:f67a1a692e5f5b96b89df0ba7ff557dfefe0ebe1161509ab71fd7d09109a4d65
    Port:          <none>
    Host Port:     <none>
    Command:
      compliance-controller
    Args:
      --watch-ns=mcm-cem-cluster-ns
      --cluster-name=mcm-cem-cluster
      -logtostderr=true
      -v=5
    State:          Running
      Started:      Fri, 22 Mar 2019 23:50:30 +1100
    Ready:          True
    Restart Count:  0
    Environment:    <none>
    Mounts:
      /var/run/secrets/kubernetes.io/serviceaccount from default-token-rk8bc (ro)
Conditions:
  Type              Status
  Initialized       True
  Ready             True
  ContainersReady   True
  PodScheduled      True
Volumes:
  weavescope-ca-secret:
    Type:        Secret (a volume populated by a Secret)
    SecretName:  mcm-klusterlet-rel-ibm-mcmk-prod-weavescope-client-secrets
    Optional:    false
  klusterlet-config:
    Type:        Secret (a volume populated by a Secret)
    SecretName:  mcm-klusterlet-rel-ibm-mcmk-prod-klusterlet
    Optional:    false
  tiller-secret:
    Type:        Secret (a volume populated by a Secret)
    SecretName:  mcm-klusterlet-rel-ibm-mcmk-prod-tiller-client-certs
    Optional:    false
  default-token-rk8bc:
    Type:        Secret (a volume populated by a Secret)
    SecretName:  default-token-rk8bc
    Optional:    false
QoS Class:       BestEffort
Node-Selectors:  <none>
Tolerations:     node.kubernetes.io/not-ready:NoExecute for 300s
                 node.kubernetes.io/unreachable:NoExecute for 300s
Events:          <none>
[root@icp3m1 ~]#

```
Notice that there are 3 containers:
- work-manager
- policy-controller
- compliance-controller
What do you think is each of their functions.

Also watch the parameter to the containers such as
- --watch-ns=mcm-cem-cluster-ns
- --cluster-name=mcm-cem-cluster

### Remote Managed Clusters
Notice the pods that made up the Remote Managed Clusters (substitute the klusterletrelease with your release name, or use `kubctl get pods | grep ibm-mcmk` )

```
[root@icp2m1 ~]# kubectl get pods -l release=klusterletrelease
NAME                                                              READY   STATUS    RESTARTS   AGE
klusterletrelease-ibm-mcmk-prod-klusterlet-7dd99b58db-t5jls       4/4     Running   0          6d10h
klusterletrelease-ibm-mcmk-prod-weave-scope-app-58d5cfc5b7m59bd   2/2     Running   0          6d10h
klusterletrelease-ibm-mcmk-prod-weave-scope-tb6wl                 1/1     Running   0          6d10h
klusterletrelease-ibm-mcmk-prod-weave-scope-xmfnz                 1/1     Running   0          6d10h
```
Note there are 4 containers running in the klusterlet pod, one more container compare to the klusterlet than run on the hub controller.

```
[root@icp2m1 ~]# kubectl describe pod klusterletrelease-ibm-mcmk-prod-klusterlet-7dd99b58db-t5jls
Name:               klusterletrelease-ibm-mcmk-prod-klusterlet-7dd99b58db-t5jls
Namespace:          kube-system
Priority:           0
PriorityClassName:  <none>
Node:               172.16.254.202/172.16.254.202
Start Time:         Wed, 27 Mar 2019 18:13:30 +1100
Labels:             app=ibm-mcmk-prod
                    chart=ibm-mcmk-prod-3.1.2
                    component=klusterlet
                    heritage=Tiller
                    pod-template-hash=7dd99b58db
                    release=klusterletrelease
Annotations:        kubernetes.io/psp: ibm-privileged-psp
                    productID: 354b8990aab44c9988a0edfda101b128
                    productName: IBM Multi-cloud Manager - Klusterlet
                    productVersion: 3.1.2
Status:             Running
IP:                 10.1.32.239
Controlled By:      ReplicaSet/klusterletrelease-ibm-mcmk-prod-klusterlet-7dd99b58db
Containers:
  work-manager:
    Container ID:  docker://a6b7a6238a231fe21d2267ecead26858a0c800986d768335a7ec0ebdc35f9255
    Image:         smallcluster.icp:8500/kube-system/mcm-klusterlet:3.1.2
    Image ID:      docker-pullable://smallcluster.icp:8500/kube-system/mcm-klusterlet@sha256:933daaadb55fa0a2c90e0e0dd9a53e0ff9d3bf7e5be6ddec6f3d84270f1c8a9f
    Port:          <none>
    Host Port:     <none>
    Args:
      /klusterlet
      --bootstrap-controller-config-file=/var/run/bootstrap-klusterlet/config
      --client-cert-secret=kube-system/klusterletrelease-ibm-mcmk-prod-klusterlet-cert
      --controller-config-file=/var/run/klusterlet/config
      --cluster-name=mcm-small-cluster
      --cluster-namespace=mcm-small-cluster-ns
      --cluster-labels=cloud=IBM,vendor=ICP,environment=Dev,region=US,datacenter=Dallas,owner=csmo
      --tiller-cert=/etc/helm/tls.crt
      --tiller-key=/etc/helm/tls.key
      --helm-release-prefix=md
      --weave-host=https://klusterletrelease-ibm-mcmk-prod-weave-scope.kube-system:443
      --weave-interval=15
      --weave-cert=/opt/ibm/router/client-certs/tls.crt
      --weave-key=/opt/ibm/router/client-certs/tls.key
      --klusterlet-ingress=kube-system/klusterletrelease-ibm-mcmk-prod-klusterlet
      --klusterlet-address=mcm-small-cluster.klusterlet.mcm
    State:          Running
      Started:      Wed, 27 Mar 2019 18:13:34 +1100
    Ready:          True
    Restart Count:  0
    Environment:    <none>
    Mounts:
      /etc/helm from tiller-secret (rw)
      /opt/ibm/router/client-certs from weavescope-ca-secret (rw)
      /var/run/bootstrap-klusterlet from klusterlet-config (rw)
      /var/run/secrets/kubernetes.io/serviceaccount from default-token-cllpf (ro)
  policy-controller:
    Container ID:  docker://2ebcd8bac349db581c60f70591039de6846eb6331c798b4a8284d888c20ed6a5
    Image:         smallcluster.icp:8500/kube-system/mcm-compliance:3.1.2
    Image ID:      docker-pullable://smallcluster.icp:8500/kube-system/mcm-compliance@sha256:f67a1a692e5f5b96b89df0ba7ff557dfefe0ebe1161509ab71fd7d09109a4d65
    Port:          <none>
    Host Port:     <none>
    Command:
      policy-controller
    Args:
      -logtostderr=true
      --watch-ns=mcm-small-cluster-ns
      --cluster-name=mcm-small-cluster
      -v=5
      --alert-target=true
    State:          Running
      Started:      Wed, 27 Mar 2019 18:13:34 +1100
    Ready:          True
    Restart Count:  0
    Environment:    <none>
    Mounts:
      /var/run/secrets/kubernetes.io/serviceaccount from default-token-cllpf (ro)
  compliance-controller:
    Container ID:  docker://c11e045ed1931cafc36316edaf75aebcd443a5e0c35c38979fddc72ee2280a46
    Image:         smallcluster.icp:8500/kube-system/mcm-compliance:3.1.2
    Image ID:      docker-pullable://smallcluster.icp:8500/kube-system/mcm-compliance@sha256:f67a1a692e5f5b96b89df0ba7ff557dfefe0ebe1161509ab71fd7d09109a4d65
    Port:          <none>
    Host Port:     <none>
    Command:
      compliance-controller
    Args:
      --watch-ns=mcm-small-cluster-ns
      --cluster-name=mcm-small-cluster
      -logtostderr=true
      -v=5
    State:          Running
      Started:      Wed, 27 Mar 2019 18:13:35 +1100
    Ready:          True
    Restart Count:  0
    Environment:    <none>
    Mounts:
      /var/run/secrets/kubernetes.io/serviceaccount from default-token-cllpf (ro)
  sync-controller:
    Container ID:  docker://3abef5850547933c155a5a52fd9c5a42315a78ab6c24ca0d0e3c17453163141c
    Image:         smallcluster.icp:8500/kube-system/mcm-compliance:3.1.2
    Image ID:      docker-pullable://smallcluster.icp:8500/kube-system/mcm-compliance@sha256:f67a1a692e5f5b96b89df0ba7ff557dfefe0ebe1161509ab71fd7d09109a4d65
    Port:          <none>
    Host Port:     <none>
    Command:
      sync-controller
    Args:
      --remote-kubeconfig=/var/run/klusterlet/config
      --watch-ns=mcm-small-cluster-ns
      --klusterlet-ns=kube-system
      --cert-secret-name=klusterletrelease-ibm-mcmk-prod-klusterlet-cert
      --alert-target=true
      -logtostderr=true
      -v=5
    State:          Running
      Started:      Wed, 27 Mar 2019 18:13:35 +1100
    Ready:          True
    Restart Count:  0
    Environment:    <none>
    Mounts:
      /var/run/klusterlet from klusterlet-config (rw)
      /var/run/secrets/kubernetes.io/serviceaccount from default-token-cllpf (ro)
Conditions:
  Type              Status
  Initialized       True
  Ready             True
  ContainersReady   True
  PodScheduled      True
Volumes:
  weavescope-ca-secret:
    Type:        Secret (a volume populated by a Secret)
    SecretName:  klusterletrelease-ibm-mcmk-prod-weavescope-client-secrets
    Optional:    false
  klusterlet-config:
    Type:        Secret (a volume populated by a Secret)
    SecretName:  klusterletrelease-ibm-mcmk-prod-klusterlet
    Optional:    false
  tiller-secret:
    Type:        Secret (a volume populated by a Secret)
    SecretName:  klusterletrelease-ibm-mcmk-prod-tiller-client-certs
    Optional:    false
  default-token-cllpf:
    Type:        Secret (a volume populated by a Secret)
    SecretName:  default-token-cllpf
    Optional:    false
QoS Class:       BestEffort
Node-Selectors:  <none>
Tolerations:     node.kubernetes.io/not-ready:NoExecute for 300s
                 node.kubernetes.io/unreachable:NoExecute for 300s
Events:          <none>
[root@icp2m1 ~]#
```
Notice there are 4 containers.  The additional container is `sync-controller`.
What do you think is its function?
