Containers are the base unit of deployment that runs any application on Kubernetes. Containers are processes that run on a given Kubernetes Host, they can have access to the host file systems, networks, host namespaces, password files, listen to traffic on the host etc. 
An application running in a container can see host/system level objects. To prevent containers from doing so, Kubernetes has created Admission Controllers that check the provisioning of a pod based on a set of Pod Security Policies (PSPs). It is important to not let containers access host based resources unless necessary as doing so opens unnecessary potential attack vectors. By default, Kubernetes does not implement any pod security policies.

Using VMware Tanzu Mission Control, you can make the deployments to your clusters more secure by implementing constraints that govern what deployed pods can do. Security policies, implemented using OPA Gatekeeper, allow you to restrict certain aspects of pod execution in your clusters, such as privilege escalation, Linux capabilities, and allowed volume types.

To create these policies:

<details>
<summary><b>TMC Console</b></summary>
<p>

* Click on the **Security** tab within the policy assignments section and click on the **Clusters** view if you are still seeing **Workspaces**.

* Click on the **tkoworkshop-w01-s001-cluster** Cluster under the Cluster Group **tkoworkshop-w01-s001-cg**

* Click **Create Security Policy**

* Select the **Security template** *Strict*

* Provide a policy name `strict-security-policy-ui`

* Click **Create policy**

</p>
</details>

<details>
<summary><b>TMC CLI</b></summary>
<p>

* Before we apply this policy using the TMC CLI, let's have a look on its definition and do some modifications

```editor:open-file
### file: ~/strict-security-policy.yaml
fullName:
  clusterName: tkoworkshop-w01-s001-cluster
  name: strict-security-policy-cli
  managementClusterName: attached
  provisionerName: attached
spec:
  input: {}
  recipe: strict
  recipeVersion: v1
  type: security-policy
```

* Create the strict security policy 

    ```execute-1
    tmc cluster security-policy create -f strict-security-policy.yaml
    ```

* Confirm that the policy has been created and synced to tkoworkshop-w01-s001-cluster 

    ```execute-1
    tmc cluster security-policy get strict-security-policy-cli --cluster-name tkoworkshop-w01-s001-cluster
    ```

    ```execute-2
    kubectl get opapolicies.intents.tmc.cloud.vmware.com --kubeconfig=.kube/config
    ```

    You should get both `strict-security-policy-cli` and `strict-security-policy-ui` listed
    
* Wait until all pods in **gatekeeper-system** Namespace are in **1/1 Ready** Status

    ```execute-2
    kubectl get pods -n gatekeeper-system
    ```    
</p>
</details>
<p>
</p>

Now we will deploy an app with root privileges on the cluster **
tkoworkshop-w01-s001-cluster** that has security policy enabled.

* Go to the workshop tab, on the Terminal Tab

```execute-1
kubectl create deployment nginx --image=nginx --kubeconfig=.kube/config -n default
```

* Check if the deployment has been created or not

```execute-1
kubectl get po --kubeconfig=.kube/config -n default
```

```execute-1
kubectl get rs --kubeconfig=.kube/config -n default
```

* Notice that the admission webhook blocks the creation due to privilege escalation being blocked:

```execute-1
kubectl get events --field-selector type=Warning --kubeconfig=.kube/config -n default --sort-by='.metadata.creationTimestamp'
```

This is because the security policy is enabled on the cluster is blocking any cluster needing privileged mode/root access implemented by Tanzu Mission Control.

* Delete the deployment

```execute-1
kubectl delete deployment nginx --kubeconfig=.kube/config -n default
```

* Now let's deploy an application that is **Strict Policy** compliant 

```execute-1
cat deployment-with-security-policy.yaml 

apiVersion: apps/v1
kind: Deployment
metadata:
  labels:
    app: kuard
  name: kuard
spec:
  replicas: 1
  selector:
    matchLabels:
      app: kuard
  template:
    metadata:
      labels:
        app: kuard
    spec:
      containers:
        - name: kurd
          image: gcr.io/kuar-demo/kuard-amd64:blue
          imagePullPolicy: IfNotPresent
          securityContext:
            allowPrivilegeEscalation: false
            runAsUser: 2000
          ports:
            - containerPort: 8080
          resources:
            requests:
              cpu: "100m"
              memory: 256Mi
            limits:
              cpu: "200m"
              memory: 512Mi

kubectl apply -f deployment-with-security-policy.yaml --kubeconfig=.kube/config -n default
```

Note the `securityContext` specs of this deployment 

```editor:select-matching-text
    securityContext:
      allowPrivilegeEscalation: false
      runAsUser: 2000
```
* Delete the deployment

    ```execute-1
    kubectl delete -f deployment-with-security-policy.yaml --kubeconfig=.kube/config -n default
    ```

* Delete the created policies  

    ```execute-1
    tmc workspace security-policy delete strict-security-policy-cli --cluster-name tkoworkshop-w01-s001-cluster
    ```

    ```execute-1
    tmc workspace security-policy delete strict-security-policy-ui --cluster-name tkoworkshop-w01-s001-cluster

    ```
