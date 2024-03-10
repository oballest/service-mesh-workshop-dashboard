# Digging into Observability 

Istio provides additional capabilities to analyze the service mesh and its performance.  Let's deploy a new version of the user profile service and analyze its effect on the service mesh.

<br>

The deployment file 'userprofile-deploy-v2.yaml' was created for you to deploy the application.

<blockquote>
<i class="fa fa-terminal"></i>
Deploy the service using the image URI:
</blockquote>

```execute
oc create -f  ./config/app/userprofile-deploy-v2.yaml
```

<blockquote>
<i class="fa fa-terminal"></i>
Watch the deployment of the user profile:
</blockquote>

```execute
oc get pods -l deploymentconfig=userprofile --watch
```

Output:
```
userprofile-2-xxxxxxxxxx-xxxxx            2/2     Running        0          22s
userprofile-xxxxxxxxxx-xxxxx              2/2     Running        0          2m55s
```

<br>

## Access Application

Let's test the new version of our profile service in the browser (spoiler: you added a bug).

<blockquote>
<i class="fa fa-desktop"></i>
Navigate to the 'Profile' section in the header.  
</blockquote>

<p><i class="fa fa-info-circle"></i> If you lost the URL, you can retrieve it via:</p>

```execute
echo $GATEWAY_URL
```

<br>

The profile page will round robin between versions 1 and 2.  Version 2 loads really slowly and looks like this:

<img src="images/app-profilepage-v2.png" width="1024"><br/>
 *Profile Page*

Next, we will use the Service Mesh to debug the problem.