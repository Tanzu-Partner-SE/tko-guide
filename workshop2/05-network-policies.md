Using VMware Tanzu Mission Control, you can create a network policy that defines how pods communicate with each other and other network endpoints, using preconfigured templates called recipes. By default, Tanzu Mission Control does not impose any such restriction, and allows you to manage network restrictions at the organizational level and at the workspace level.

Tanzu Mission Control implements network policies using Kubernetes native network policies. Each namespace and workspace can be governed by a network policy, and these policies are inherited down through the organizational hierarchy. Network policies are additive, both inherited and direct network policies are applied and are effective on your namespaces according to Kubernetes rules.

To create a network policy for an object, you must be associated 
with the *.admin* role for that object.

Let us follow the following procedure to create a network policy that allows all ingress traffic in 
our workspace  **tkoworkshop-w01-s001-ws**:

1. On the Policies page, click the Network tab, and then click the Workspaces organization view.

2. Use the tree control to navigate to and select the object for which you want to create a network policy.  
In this case, click **tkoworkshop-w01-s001-ws** to add a network policy for this workspace.

  ![](./images/policy-network-1.png)

3. Click Create Network Policy.

4. Select the network policy recipe to use. Choose one of the values from the dropdown. Available values 
are:

- *`allow-all`* (allow all ingress traffic)
- *`deny-all`* (deny all ingress traffic)
- *`allow-all-to-pods`* (allow all ingress traffic to selected pods)
- *`deny-all-to-pods`* (deny all ingress traffic to selected pods)
- *`custom-ingress`* (define a custom ingress policy)
- *`custom-egress`* (define an egress policy). In this case, we select *allow-all*.


5. Provide a policy name: `allow-all-policy`{{copy}}.

6. Click Create Policy.

  ![](./images/policy-network-allow-all.png) 

Now let's create a `custom-ingress` policy and test it. This policy will allow ingress network connection to a `web-server` POD from `allowed-client` POD only. 

Before we apply this policy using the TMC CLI, let's have a look at its definition:

```
### file: ~/network-policy.yaml
fullName:
  name: custom-ingress
  workspaceName: tkoworkshop-w01-s001-ws:
spec:
  input:
    rules:
      - ports:
          - port: "80"
            protocol: TCP
        ruleSpec:
          - podSelector:
              - key: app
                value: allowed-client
    toPodLabels:
      - key: app
        value: web-server
  recipe: custom-ingress
  recipeVersion: v1
  type: network-policy
```

* Create a policy 

    ```execute-1
    tmc workspace network-policy create -f network-policy.yaml 
    ```

* Confirm that the policy has been created and synced to the tkoworkshop-w01-s001-cluster:

    ```execute-1
    tmc workspace network-policy get custom-ingress --workspace-name tkoworkshop-w01-s001-ws
    ```

    ```execute-1
    kubectl describe networkpolicies --kubeconfig=.kube/config tmc.wsp.tkoworkshop-w01-s001.custom-ingress -n tkoworkshop-w01-s001
    ```

Let's validate that our network policy is working by trying to deploy three PODs (`web-server`, `allowed-client` and `not-allowed-client`) to the namespace **tkoworkshop-w01-s001**,
which is part of the workspace **tkoworkshop-w01-s001-ws**. 

* Deploy the test PODs:

```execute-1
kubectl --kubeconfig=.kube/config apply -f network-policy-deployment/ -n tkoworkshop-w01-s001
```

* Confirm that the pods are up and running

```execute-1
kubectl --kubeconfig=.kube/config get po -n tkoworkshop-w01-s001
```

* Run `curl command` from `not-allowed-client` POD to test the connection to the `web-server` POD.

```execute-1
kubectl --kubeconfig=.kube/config exec -n tkoworkshop-w01-s001 deploy/not-allowed-client -- curl http://web-server:80 --connect-timeout 3 -s
```

You should receive **command terminated with exit code 28** Error

* Run `curl command` from `allowed-client` POD to test the connection to the `web-server` POD

```execute-1
kubectl --kubeconfig=.kube/config exec -n tkoworkshop-w01-s001 deploy/allowed-client -- curl http://web-server:80 --connect-timeout 3 -s
```

As you can see, our policy allows the connection between `allowed-client` POD and `web-server` POD

* Delete the test PODs

```execute-1
kubectl --kubeconfig=.kube/config delete -f network-policy-deployment/ -n tkoworkshop-w01-s001
```

* Delete the created policy 

```execute-1
tmc workspace network-policy delete custom-ingress --workspace-name tkoworkshop-w01-s001-ws
```
