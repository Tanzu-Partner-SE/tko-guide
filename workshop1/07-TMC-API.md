In this section, lets explore few rest API's. you can find complete list of API's in:

```dashboard:open-url
url: https://developer.vmware.com/apis/897/tanzu-mission-control
```

<style>
H5{color:Blue !important;}
H6{color:DarkOrange !important;}
</style>

<p style="color:blue"><strong> Fetch API Token:  </strong></p> 

<p style="color:red"><strong> Note: In Terminal-1, just right click and paste to edit above curl command. replace the API Token.  </strong></p> 

```copy-and-edit
refresh_token=<replace me with TMC API Token>
```

<p style="color:black"><strong>Example:</strong></p>

![TMC Token](images/TMC-token.png)

<p style="color:blue"><strong> Generate Access token from above Refresh Token </strong></p> 

```execute
access_token=$(curl -d "refresh_token=$refresh_token" https://console.cloud.vmware.com/csp/gateway/am/api/auth/api-tokens/authorize | jq -r '.access_token')
```

<p style="color:blue"><strong> Read the Access Token </strong></p> 

```execute
echo $access_token
```

<p style="color:blue"><strong> Verify the access(Optional) using access token </strong></p> 

```dashboard:open-url
url: https://jwt.io/
```

<p style="color:black"><strong>Example:</strong></p>

![TMC Token JWT](images/TMC-token2.png)

<p style="color:blue"><strong> Get the organization ID and account details  </strong></p> 

```execute
curl -s --request POST --url https://console.cloud.vmware.com/csp/gateway/am/api/auth/api-tokens/details --header 'content-type: application/json' --data '{"tokenValue":"'"$refresh_token"'"}' | jq '.'
```

<p style="color:blue"><strong> Get the cluster groups </strong></p> 

```execute
curl -s 'searchScope.name=*' https://partnertanzuseamericas.tmc.cloud.vmware.com/v1alpha1/clustergroups -H "Authorization: Bearer $access_token" | jq '.clusterGroups[].fullName.name'
```

<p style="color:blue"><strong>  Get the clusters  </strong></p> 

```execute
curl -s 'searchScope.name=*' https://partnertanzuseamericas.tmc.cloud.vmware.com/v1alpha1/clusters -H "Authorization: Bearer $access_token" | jq '.clusters[].fullName.name'
```

<p style="color:blue"><strong> Get the management clusters  </strong></p> 

```execute
curl -s 'searchScope.name=*' https://partnertanzuseamericas.tmc.cloud.vmware.com/v1alpha1/managementclusters -H "Authorization: Bearer $access_token" | jq '.managementClusters[].fullName.name'
```

<p style="color:blue"><strong> Delete the cluster group {{ session_namespace }}-cg that is created earlier:  </strong></p> 

```execute
curl -X DELETE https://partnertanzuseamericas.tmc.cloud.vmware.com/v1alpha1/clustergroups/$SESSION_NAME-cg -H "Authorization: Bearer $access_token"
```

<p style="color:blue"><strong> See the cluster groups again and now you should see that {{ session_namespace }}-cg is missing from the list </strong></p> 

```execute
curl -s 'searchScope.name=*' https://partnertanzuseamericas.tmc.cloud.vmware.com/v1alpha1/clustergroups -H "Authorization: Bearer $access_token" | jq '.clusterGroups[].fullName.name'
```
