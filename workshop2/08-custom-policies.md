For many aspects of cluster management, VMware Tanzu Mission Control provides specific policies that you can use to enforce rules on your fleet of Kubernetes clusters, such as access policies, image registry policies, and namespace quota policies. However, not all of the aspects that you might want to control are covered by these baseline policies. Custom policies are somewhat open-ended and provide the opportunity to address aspects of cluster management that specifically suit the needs of your organization.

To implement a custom policy, you must first have a template that declaratively defines the structure of the policy. The custom policy template can then be used to create and apply custom policies to your clusters.

Custom policies in Tanzu Mission Control are implemented using the Gatekeeper project from Open Policy Agent (OPA). To create a custom policy template, you use Rego, the policy language of OPA. For more information about Rego, see the OPA [Policy Language](https://www.openpolicyagent.org/docs/latest/policy-language/) documentation.

<details>
<summary><b>TMC Console</b></summary>
<p>

* On the Policies page, click the Custom tab, and then click the Clusters organization view
* Use the tree control to navigate to **{{ session_namespace }}-cluster** Cluster under the Cluster Group **tko-day1-ops** 
* Click Create Custom Policy
* On the custom policy create form, select the policy template **tmc-require-labels** , and then provide a name `{{ session_namespace }}-rl-ui`{{copy}} for the policy
* Specify the target resources *Deployment*, and then click **Add Resource**
* Specify parameters for your policy, under **Key** add **env**
* Click **Create Policy**

Now we will deploy an app without the required tags on the cluster **{{ session_namespace }}-cluster** .

* Let's deploy an app that doesn't have the required label key

```execute-1
kubectl apply -f deployment-without-tags.yaml --kubeconfig=.kube/config -n default
```
```execute-1
kubectl get po --kubeconfig=.kube/config -n default
```
```execute-1
kubectl get rs --kubeconfig=.kube/config -n default
```
* Notice that the admission webhook blocks the creation due to the missing tags in the deployment metadata:

```execute-1
kubectl get events --field-selector type=Warning --kubeconfig=.kube/config -n default --sort-by='.metadata.creationTimestamp'
```
```execute-1
kubectl delete -f deployment-without-tags.yaml --kubeconfig=.kube/config -n default
```
* Let's deploy an app that complies our tag policy

```execute-1
kubectl apply -f deployment-with-tags.yaml --kubeconfig=.kube/config -n default
```
```execute-1
kubectl get po --kubeconfig=.kube/config -n default
```

* Cleanup the created resources and policy 

```execute-1
kubectl delete -f deployment-with-tags.yaml --kubeconfig=.kube/config -n default
```
```execute-1
tmc cluster custom-policy delete {{ session_namespace }}-rl-ui --cluster-name {{ session_namespace }}-cluster 
```