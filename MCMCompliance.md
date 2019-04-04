# MCM Compliance

## Logging in

Start by login to the cluster if you have not done so.

To make the exercise easier to execute, open two terminal windows, one at the hub controller and another one at the remote clusters.  Login to each clusters in the corresponding terminal windows.

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

## Getting your local information.
you can view the clusters:
```
JuliusMBP:~ jwahidin$ kubectl get clusters --all-namespaces
NAMESPACE              NAME                ENDPOINTS             STATUS   AGE
mcm-cem-cluster-ns     mcm-cem-cluster     172.16.254.88:8001    Ready    12d
mcm-small-cluster-ns   mcm-small-cluster   172.16.254.203:8001   Ready    8d
```

Note the namespaces and the cluster name, you will need to replace the yaml file with the namespace and cluster name of your environment.

## Useful command set
The following command will be useful for this exercise, you might want to copy and paste to execute the command:
```
kubectl get compliances --all-namespaces
kubectl get policies --all-namespaces
kubectl get role --your klusterlet name space
kubectl get limitrange --your klusterlet name space

```
First, check your current environment.  At the hub controller:
```
[root@icp1m1 compliance]# kubectl get compliances --all-namespaces
No resources found.
[root@icp1m1 compliance]# kubectl get policies --all-namespaces
No resources found.
[root@icp1m1 compliance]# kubectl get role --all-namespaces
NAMESPACE     NAME                                                      AGE
kube-public   ibmcloud-cluster-info                                     24d
kube-public   system:controller:bootstrap-signer                        24d
kube-system   extension-apiserver-authentication-reader                 24d
kube-system   servicecatalog.k8s.io:cluster-info-configmap              24d
kube-system   servicecatalog.k8s.io:leader-locking-controller-manager   24d
kube-system   system::leader-locking-kube-controller-manager            24d
kube-system   system::leader-locking-kube-scheduler                     24d
kube-system   system:controller:bootstrap-signer                        24d
kube-system   system:controller:cloud-provider                          24d
kube-system   system:controller:token-cleaner                           24d
mcm-main-ns   hub-maincluster                                           12d
smallns       smallcluster                                              12h
[root@icp1m1 compliance]# kubectl get limitrange --all-namespaces
No resources found.

```
Note there should not be any compliance and policy.

## Exercise 1: Test inform on the local cluster.
We want to check if the current cluster has operator role.  View the c1.yaml then apply it.
```
kubectly apply -f c1.yaml
```

We are making use of the watch-ns for this exercise. So no placement policy and placement binding are required.
Check the result:
```
[root@icp1m1 compliance]# kubectl get compliances --all-namespaces
NAMESPACE     NAME     AGE
mcm-main-ns   testc1   1m
[root@icp1m1 compliance]# kubectl get policies --all-namespaces
NAMESPACE     NAME       AGE
mcm-main-ns   policy01   1m
```
Note the compliance and policy are created.

Check the result by running a `kubectl describe`.

```
[root@icp1m1 compliance]# kubectl describe policies policy01 -n mcm-main-ns

[root@icp1m1 compliance]# kubectl describe compliances testc1 -n mcm-main-ns

```
Observe the output.

Once you are ok, remove the compliance `kubectl delete compliance testc1 -n mcm-main-ns`

## Exercise 2: Test inform on the remote cluster.

In this exercise we want to check the compliance on the remote cluster from the hub-cluster.
Use c2.yaml to do this, and repeat the command above.
You might want to run some of the verification from the remote cluster.
Delete the compliance and policy before moving to the next exercise.

## Exercise 3: Test enforce on the local cluster.
In this exercise we want mcm to perform action if the environment does not comply through the enforce.
Apply c3.yaml
Note that the Role and Memory Limit are created.
Again delete the compliance, role and limit before moving to the next exercise.

## Exercise 4: Test enforce on a remote cluster.
Apply c4.yaml
Note that the role and Memory Limite should be enforced in the remote cluster.

Note while applying c1 to c4 yaml, we do not make use of the namespace of the MCM itself, as a result the compliance and policy is not displayed in the GUI.

You have to use the MCM namespace that you specify during install in the yaml file for the compliance and policy to appear.  Starting in version 3.1.2 you also need to create the placement policy and placement binding.
