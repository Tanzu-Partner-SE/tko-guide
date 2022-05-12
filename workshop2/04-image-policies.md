Using VMware Tanzu Mission Control, you can make the deployments to namespaces in your clusters more secure by restricting the image registries from which images can be pulled, as well as the images that can be pulled from a registry. By default, Tanzu Mission Control does not impose any such restriction, and allows you to manage image registry restrictions at the organizational level and at the workspace level.

Tanzu Mission control has Image based policies that can be applied to namespaces within a cluster. These policies can be applied fleet-wide across clusters and clouds by grouping namespaces together in a logical group called Workspaces.

Each namespace and workspace can be protected by an image registry policy that defines the registries from which an image can be pulled, and these policies are inherited down through the organizational hierarchy.

Examples of image registry polices include:

- Implement Policies to not allow images from certain Image Registries
- Policy that prevents container images with no digest from deploying
- Stop container images with latest tag from deploying 
- Blacklist certain images/repos 

<div class="info" style='background-color:#e7f3fe; color: #000000; border-left: solid #2196F3 4px; border-radius: 4px; padding:0.7em;'>
<span>
<p style='margin-top:1em; text-align:left'>
<b>Note:</b></p>
<p style='margin-left:1em;'>
Image registry policies cannot be set at a cluster level. Image policies can be assigned only at workspace level or at the organization level.
</p>
</span>
</div>
<p>
</p>

To create an image registry policy for an object, you must be associated with the **.admin** role for that object

* Click Create Image Registry Policy.
* Select the recipe you want to use.
    * The *`Block latest tag`* recipe prevents the use of images that are tagged latest.
    * The *`Require Digest`* recipe prevents the use of images that do not have a digest.
    * The *`Name-Tag allowlist`* recipe allows you to create rules using an image name or tag name or both.
    * The *`Custom`* recipe allows you to create rules using multiple factors.

As quick example we are going to create a *`Require Digest`* recipe policy using the TMC Console

![](./images/policy-image-registry-1.png)

Select your workspace ***{{ session_namespace }}-ws*** and click Create Image 
Registry Policy. We are going to configure that all the container 
images to be deployed to this workspace must have a container digest. 
Under Image registry template dropdown, select Require Digest:

![](./images/policy-image-registry-digest-1.png)

Give it a name such as `{{ session_namespace }}-di-policy`{{copy}} and proceed with default values 
for other fields. If needed, you may specify label selectors to 
include or exclude certain namespaces for this policy. 
Finally click Create Policy. 

The steps above show how to create an image registry policy to require digest. Similarly, image 
registry policies to specify a name-tag allowlist, block the latest tag, or even custom policies may be created:
 
![](./images/policy-image-registry-digest-2.png)

Once created, you may edit or delete an image registry policy.

Let's validate that our image *`Require Digest`* registry policy is working by trying to deploy a container image with and without a digest to the namespace **{{ session_namespace }}**
 
* Confirm that the policy has been created    
    
    ```execute-1
    tmc workspace image-policy get tko-tmc-day1-w01-s001-di-policy  --workspace-name tko-tmc-day1-w01-s001-ws
    ```
    ```execute-1
    kubectl describe opapolicies.intents.tmc.cloud.vmware.com --kubeconfig=.kube/config wsp.tko-tmc-day1-w01-s001.tko-tmc-day1-w01-s001-di-policy.vmware-system-tmc-allowed-images-v1
    ```
* Wait until all pods in **gatekeeper-system** Namespace are in **Ready** Status

    ```execute-1
    kubectl get pods -n gatekeeper-system
    ```
* Create a deployment with **nginx** image:

```execute-1
kubectl --kubeconfig=.kube/config create deployment nginx-without-digest --image=nginx -n tko-tmc-day1-w01-s001
```

* Notice the deployment is blocked and won't progress because of *`Require Digest`* registry policy.

```execute-1
kubectl --kubeconfig=.kube/config get events --field-selector type=Warning -n tko-tmc-day1-w01-s001 --sort-by='.metadata.creationTimestamp'
```

* Confirm that the nginx pod hasn't been deployed

```execute-1
kubectl --kubeconfig=.kube/config get pods -n tko-tmc-day1-w01-s001
```

* Delete the deployment

```execute-1
kubectl --kubeconfig=.kube/config delete deployment nginx-without-digest -n tko-tmc-day1-w01-s001
```

* Now let's deploy a nginx container with digest to check if the policy will allow it run

```execute-1
kubectl --kubeconfig=.kube/config create deployment nginx-with-digest --image=nginx@sha256:2275af0f20d71b293916f1958f8497f987b8d8fd8113df54635f2a5915002bf1 -n tko-tmc-day1-w01-s001
```

* Confirm that the nginx pod with the image digest has been deployed

```execute-1
kubectl --kubeconfig=.kube/config get pods -n tko-tmc-day1-w01-s001
```

* Again, check the events if there is any error, ignore the errors from the previous test


```execute-1
kubectl --kubeconfig=.kube/config get events --field-selector type=Warning -n tko-tmc-day1-w01-s001 --sort-by='.metadata.creationTimestamp'
```

* Delete the deployment

```execute-1
kubectl --kubeconfig=.kube/config delete deployment nginx-with-digest -n tko-tmc-day1-w01-s001
```

* Delete the created policy 

```execute-1
tmc workspace image-policy delete tko-tmc-day1-w01-s001-di-policy --workspace-name tko-tmc-day1-w01-s001-ws
```


Now let's create a custom policy in workspace ***ko-tmc-day1-w01-s020-ws*** that blocks any container image that doesn't have the name `busybox`: 

<div class="info" style='background-color:#e7f3fe; color: #000000; border-left: solid #2196F3 4px; border-radius: 4px; padding:0.7em;'>
<span>
<p style='margin-top:1em; text-align:left'>
<b>Note:</b></p>
<p style='margin-left:1em;'>
When you define a custom policy, you can use wildcards to match specific patterns, for example:
- Registry Hostname (*.mycompany.com)
- Image name (fooservice/*)
- Image tag (v1.*)
</p>
</span>
</div>
<p>
</p>
<details>
<summary><b>TMC Console</b></summary>
<p>

1. Click Workspaces under the Image Registry tab in the Policies page and select workspace 
***ko-tmc-day1-w01-s001-ws***

2. Click Create Image Registry Policy

  ![](./images/policy-image-registry-custom-1.png)

3. Choose Custom in the Image Registry Template field and give it a name 
  such as `tko-tmc-day1-w01-s001-ip-ui` in the Policy Name field. Under the Rule pane, 
type in `library/busybox` in the Image Name field. Optionally, you may specify the hostname and port to restrict where the images are pulled from. In addition, you may add more rules by clicking Add Another Rule.

  ![](./images/policy-image-registry-custom-2.png)

4. Optionally, this custom rule may be made to apply to certain namespaces of this workspace if desired by specifying the Label Selectors fields. At the end, click Create Policy.
</p>
</details>
<p>
</p>
<details>
<summary><b>TMC CLI</b></summary>
<p>

Before we apply this policy using the TMC CLI, let's have a look on its yaml definition:

```execute-1
### file: ~/busybox-image-policy.yaml
fullName:
  name: busybox-ip-cli
  workspaceName: tko-tmc-day1-w01-s001-ws 
spec:
  input:
    rules:
      - imageName: library/busybox
        tag: {}
  recipe: custom
  recipeVersion: v1
  type: image-policy
```

* Create the image policy 

    ```execute-1
    tmc workspace image-policy create -f busybox-image-policy.yaml 
    ```
* Confirm that the image policy has been created and synced to the **tko-tmc-day1-w01-s020-cluster**:

    ```execute-1
    tmc workspace image-policy get busybox-ip-cli --workspace-name tko-tmc-day1-w01-s001-ws
    ```

    ```execute-1
    kubectl describe opapolicies.intents.tmc.cloud.vmware.com --kubeconfig=.kube/config wsp.tko-tmc-day1-w01-s001.busybox-ip-cli.vmware-system-tmc-allowed-images-v1
    ```
* Repeat the previous command until the policy **Status** changes to **True**

* Wait until all pods in **gatekeeper-system** Namespace are in **1/1 Ready** Status

    ```execute-2
    kubectl get pods -n gatekeeper-system
    ```
</p>
</details>
<p>
</p>

<div class="info" style='background-color:#e7f3fe; color: #000000; border-left: solid #2196F3 4px; border-radius: 4px; padding:0.7em;'>
<span>
<p style='margin-top:1em; text-align:left'>
<b>Note:</b></p>
<p style='margin-left:1em;'>
Sometimes it take a few seconds for the policy to be effective. If the test below fails, please delete the deployment and redeploy again.   
</p>
</span>
</div>
<p>
</p>

Let's validate that our image registry policy is working by trying to deploy the busybox image and non-busybox image to the namespace **tko-tmc-day1-w01-s001**, 
which is part of the workspace **tko-tmc-day1-w01-s001-ws**. 

* Create a deployment with **nginx** image:

```execute-1
kubectl --kubeconfig=.kube/config create deployment nginx --image=nginx -n tko-tmc-day1-w01-s001
```

* Confirm that the nginx pod hasn't been deployed

```execute-1
kubectl --kubeconfig=.kube/config get pods -n tko-tmc-day1-w01-s001
```

* Notice the deployment is blocked and won't progress because of the image policy that allows only busybox image name to be deployed

```execute-1
kubectl --kubeconfig=.kube/config get events --field-selector type=Warning -n tko-tmc-day1-w01-s001 --sort-by='.metadata.creationTimestamp'
```

* Delete the deployment

```execute-1
kubectl --kubeconfig=.kube/config delete deployment nginx -n tko-tmc-day1-w01-s001
```


* Now let's deploy a busybox container to check if the policy will allow it run

```execute-1
kubectl --kubeconfig=.kube/config apply -f busybox-deployment.yaml -n tko-tmc-day1-w01-s001
```
* Confirm that the busybox pod has been deployed

```execute-1
kubectl --kubeconfig=.kube/config get pods -n tko-tmc-day1-w01-s001
```
* Again, check the events if there is any error

```execute-1
kubectl --kubeconfig=.kube/config get events --field-selector type=Warning -n tko-tmc-day1-w01-s001 --sort-by='.metadata.creationTimestamp'
```

* Delete the created policy 

```execute-1
tmc workspace image-policy delete busybox-ip-cli --workspace-name tko-tmc-day1-w01-s001-ws
```
* Delete the busybox deployment

```execute-1
kubectl --kubeconfig=.kube/config delete -f busybox-deployment.yaml -n tko-tmc-day1-w01-s001
```

Now, let's create a policy that will allow pulling images from a particular container registry only


<details>
<summary><b>TMC CLI</b></summary>
<p>

Before we apply this policy using the TMC CLI, let's have a look on its definition:

```execute-1
### file: ~/registry-hotsname-policy.yaml
fullName:
  name: registry-hotsname-policy
  workspaceName: tko-tmc-day1-w01-s001-ws
spec:
  input:
    rules:
      - hostname: harbor.example.com
        tag: {}
  recipe: custom
  recipeVersion: v1
  type: image-policy
```

* Create a policy 

    ```execute-1
    tmc workspace image-policy create -f registry-hotsname-policy.yaml
    ```

* Confirm that the policy has been created    

    ```execute-1
    tmc workspace image-policy get registry-hotsname-policy --workspace-name tko-tmc-day1-w01-s001-ws 
    ```

    ```execute-1
    kubectl describe opapolicies.intents.tmc.cloud.vmware.com --kubeconfig=.kube/config wsp.tko-tmc-day1-w01-s001.registry-hotsname-policy.vmware-system-tmc-allowed-images-v1
    ```

* Repeat the previous command until the policy **Status** changes to **True**  

* Wait until all pods in **gatekeeper-system** Namespace are in **1/1 Ready** Status

    ```execute-2
    kubectl get pods -n gatekeeper-system
    ```

* Create a deployment with **nginx** image from **docker hub**:

    ```execute-1
    kubectl --kubeconfig=.kube/config create deployment nginx --image=nginx -n tko-tmc-day1-w01-s001
    ```

* Confirm that the nginx pod hasn't been deployed

    ```execute-1
    kubectl --kubeconfig=.kube/config get pods -n tko-tmc-day1-w01-s001
    ```

* Notice the deployment is blocked and won't progress because of the registry rules.

    ```execute-1
    kubectl --kubeconfig=.kube/config get events --field-selector type=Warning -n tko-tmc-day1-w01-s001 --sort-by='.metadata.creationTimestamp'
    ```

* Delete the deployment

    ```execute-1
    kubectl --kubeconfig=.kube/config delete deployment nginx -n tko-tmc-day1-w01-s001
    ```

* Delete the created policy 

    ```execute-1
    tmc workspace image-policy delete registry-hotsname-policy --workspace-name tko-tmc-day1-w01-s001-ws
    ```
</p>
</details>
<p>
</p>
