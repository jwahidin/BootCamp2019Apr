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


Here is some sample output:

```
----------- at the hub cluster -------------------

[root@icp1m1 compliance]# kubectl get compliances --all-namespaces
NAMESPACE   NAME     AGE
smallns     testc2   40s
[root@icp1m1 compliance]# kubectl get policies --all-namespaces
No resources found.
[root@icp1m1 compliance]# kubectl get role -n smallns
NAME           AGE
smallcluster   12h



----------- at the remote cluster -------------------
[root@icp2m1 ~]# k get compliances --all-namespaces
No resources found.
[root@icp2m1 ~]# kubectl get compliances --all-namespaces
NAMESPACE   NAME     AGE
smallns     testc2   1m
[root@icp2m1 ~]# kubectl get policies --all-namespaces
NAMESPACE   NAME       AGE
smallns     policy01   1m
[root@icp2m1 ~]# kubectl get role -n smallns
No resources found.
[root@icp2m1 ~]#

```
Checking the compliance itself:
```
[root@icp2m1 ~]# k describe compliances testc2 -n smallns
Name:         testc2
Namespace:    smallns
Labels:       <none>
Annotations:  ownerUID: 43549e46-4969-11e9-a8a3-005056a5e832
              seed-generation: 3
API Version:  compliance.mcm.ibm.com/v1alpha1
Kind:         Compliance
Metadata:
  Creation Timestamp:  2019-03-18T10:33:26Z
  Finalizers:
    compliance.finalizer.mcm.ibm.com
    sync.finalizer.mcm.ibm.com
  Generation:        1
  Resource Version:  3250865
  Self Link:         /apis/compliance.mcm.ibm.com/v1alpha1/namespaces/smallns/compliances/testc2
  UID:               4357fa62-4969-11e9-8d55-005056a59009
Spec:
  Runtime - Rules:
    API Version:  policy.mcm.ibm.com/v1alpha1
    Kind:         Policy
    Metadata:
      Creation Timestamp:  <nil>
      Name:                policy01
    Spec:
      Namespaces:
        Exclude:
          kube*
        Include:
          *

          Remediation Action:  inform
                Role - Templates:
                  API Version:      roletemplate.mcm.ibm.com/v1alpha1
                  Compliance Type:  musthave
                  Kind:             RoleTemplate
                  Metadata:
                    Creation Timestamp:  <nil>
                    Name:                operator
                  Rules:
                    Compliance Type:  musthave
                    Policy Rule:
                      API Groups:
                        extensions
                        apps
                      Resources:
                        deployments
                      Verbs:
                        get
                        list
                        watch
                        create
                        delete
                        patch
                  Status:
                    Validity:
              Status:
          Status:
            Status:
              Smallcluster:
                Aggregate Policies Status:
                  Policy 01:
                    Compliant:  NonCompliant
                    Valid:      true
                Clustername:    smallcluster
                Compliant:      NonCompliant
          Events:               <none>


```

Delete the compliance and policy before moving to the next exercise.

You might notice the following:
- Deleting the compliance on the remote cluster, will only recreate it.
- The compliance need to be deleted on the management cluster.

## Exercise 3: Test enforce on the local cluster.
In this exercise we want mcm to perform action if the environment does not comply through the enforce.
Apply c3.yaml
Note that the Role and limitrange are created.

```
[root@icp1m1 compliance]# kubectl get compliances --all-namespaces
NAMESPACE     NAME     AGE
mcm-main-ns   testc3   1m

[root@icp1m1 compliance]# kubectl get policies --all-namespaces
NAMESPACE     NAME       AGE
mcm-main-ns   policy01   1m
mcm-main-ns   policy02   1m

[root@icp1m1 compliance]# kubectl get role -n mcm-main-ns
NAME              AGE
hub-maincluster   12d
operator          75s

[root@icp1m1 compliance]# kubectl get limitrange -n mcm-main-ns
NAME              CREATED AT
mem-limit-range   2019-03-18T10:41:06Z

```

Check the detail of the role, limitrange and compliance itself.
```
[root@icp1m1 compliance]# kubectl describe role operator -n mcm-main-ns
Name:         operator
Labels:       <none>
Annotations:  <none>
PolicyRule:
  Resources               Non-Resource URLs  Resource Names  Verbs
  ---------               -----------------  --------------  -----
  deployments.apps        []                 []              [get list watch]
  deployments.extensions  []                 []              [get list watch]

[root@icp1m1 compliance]# kubectl describe limitrange mem-limit-range -n mcm-main-ns
Name:       mem-limit-range
Namespace:  mcm-main-ns
Type        Resource  Min  Max  Default Request  Default Limit  Max Limit/Request Ratio
----        --------  ---  ---  ---------------  -------------  -----------------------
Container   memory    -    -    256Mi            512Mi          -

[root@icp1m1 compliance]# kubectl describe compliance testc3 -n mcm-main-ns | tail -12
  Status:
    Hub - Maincluster:
      Aggregate Policies Status:
        Policy 01:
          Compliant:  Compliant
          Valid:      true
        Policy 02:
          Compliant:  NonCompliant
          Valid:      true
      Clustername:    hub-maincluster
      Compliant:      NonCompliant
Events:               <none>

```

Again delete the compliance, role and limitrange before moving to the next exercise.

## Exercise 4: Test enforce on a remote cluster.
Apply c4.yaml
Note that the role and Memory Limit should be enforced in the remote cluster.

## MCM namespace for GUI
Note while applying c1 to c4 yaml, we do not make use of the namespace of the MCM itself, as a result the compliance and policy is not displayed in the GUI.

We need to find out the namespace used.  One way to do this is do the following command:

```
JuliusMBP:~ jwahidin$ kubectl get clusterrolebinding | grep -v system | grep controller
atcrelease-alerttargetcontroller-alerttargetcontroller                 11d
mcm-release:controller:work-manager                                    13d
mcm-release:mcm:controller                                             13d
servicecatalog.k8s.io:controller-manager                               14d
```

On this command you can see that the namespace is `mcm`

Your compliance yaml should be configured for the `mcm` namespace.

You have to use the MCM namespace that you specify during install in the yaml file for the compliance and policy to appear.  Starting in version 3.1.2 you also need to create the placement policy and placement binding.
