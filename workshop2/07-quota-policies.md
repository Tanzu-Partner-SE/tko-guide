Application development teams love Kubernetes 
because they can request infrastructure resources like compute, network and storage for running their apps 
without having to deal with Operations team or raise a ticket to provision things. 

On the flip side, this means the teams managing the platform need to be aware of the capacity they have and implement any quota/restrictions on consumption. Tanzu Mission Control's Quota based policy allows you to do just that from an operations perspective.

<details>
<summary><b>TMC Console</b></summary>
<p>

* Go to the tab with Tanzu Mission Control, click on Policies then Assignments
* Click on the tab **Quota**, select CLUSTERS then click on Cluster Group >  **tkoworkshop-w01-s001-cg** > **tkoworkshop-w01-s001-cluster**
* Click on CREATE QUOTA POLICY
* Select the **Quota policy** *Small*
* Provide a policy name `small-qp-ui`

<div class="info" style='background-color:#e7f3fe; color: #000000; border-left: solid #2196F3 4px; border-radius: 4px; padding:0.7em;'>
<span>
<p style='margin-top:1em; text-align:left'>
<b>Note:</b></p>
<p style='margin-left:1em;'>

Notice it has been assigned a quota to requests of 0.5 vCPU / 512 MB memory and limit of 1 vCPU / 2 GB of memory per workload.

</p>
</span>
</div>

</p>
</details>

<details>
<p>
</p>
<summary><b>TMC CLI</b></summary>
<p>

First have a look at this quota definition file:

```editor:open-file
### file: ~/small-quota-policy.yaml
fullName:
  clusterName: tkoworkshop-w01-s001-cluster
  managementClusterName: attached
  name: small-qp-cli
  provisionerName: attached
spec:
  recipe: small
  type: namespace-quota-policy
```

* Create a policy 

    ```execute-1
    tmc cluster namespace-quota-policy create -f small-quota-policy.yaml 
    ```

* Confirm that the policy has been created and synced to the  **tkoworkshop-w01-s001-cluster**

    ```execute-1
    tmc cluster namespace-quota-policy get small-qp-cli --cluster-name tkoworkshop-w01-s001-cluster
    ```

    ```execute-1
    kubectl describe resourcequota tmc.cp.small-qp-cli --kubeconfig=.kube/config -n default
    ```
</p>
</details>

<p>

</p>

Now we will deploy an app with with request and limits on the cluster **tkoworkshop-w01-s001-cluster** that has quota policy enabled.

* Go to the workshop tab, on the Terminal Tab

```execute-1
kubectl apply -f kurd.yaml --kubeconfig=.kube/config -n default
```

* Confirm that the deployment is up and running

```execute-1
kubectl get po -n default --kubeconfig=.kube/config
```

See the updated settings on the namespace and note the data displayed in the “used” column. You will now notice a difference, with the new pod just created having used some of the quota:

```execute-2
kubectl describe resourcequota tmc.cp.small-qp-cli --kubeconfig=.kube/config -n default
```
To continue the process, we will scale our deployment to 3 replicas:

```execute-1
kubectl scale deployment kuard --replicas=3 --kubeconfig=.kube/config -n default
```
* We will see that only two pods are running although we wanted to scale our deployment to three pods

```execute-1
kubectl get po -n default --kubeconfig=.kube/config
```

Check the consumed quota again
```execute-2
kubectl describe resourcequota tmc.cp.small-qp-cli --kubeconfig=.kube/config -n default
```
* We should receive an error message states that we don’t have enough quota left to create the new pod:

```execute-1
kubectl get events --field-selector type=Warning --kubeconfig=.kube/config -n default --sort-by='.metadata.creationTimestamp'
```
* Cleanup the created resources and policy 

```execute-1
kubectl delete -f kurd.yaml --kubeconfig=.kube/config -n default
```
```execute-1
tmc cluster namespace-quota-policy delete small-qp-cli --cluster-name tkoworkshop-w01-s001-cluster
```


You can also opt to create a custom policy if you don't want to use any of the pre-defined ones or you wish to implement more detailed policies on objects such as: CPU, memory, storage, or even limits on most Kubernetes objects within a namespace.

