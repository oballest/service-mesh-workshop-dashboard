# Splitting Traffic Amongst Service Versions

It's time to fix the performance issue of the application.  Previously, you deployed a new version of the application and routed 100% of traffic to the new version.  This time, you'll use Istio traffic routing to do canary rollouts and split traffic.

## Feature Fix

The code to fix the performance issue of the user profile service has already been written for you on the 'workshop-feature-fix' branch.  


<blockquote>
<i class="fa fa-terminal"></i>
Deploy the service using the image URI:
</blockquote>

```execute
oc create -f ./config/app/userprofile-deploy-v3.yaml
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
userprofile-3-xxxxxxxxxx-xxxxx              2/2     Running        0          53s
userprofile-2-xxxxxxxxxx-xxxxx              2/2     Running        0          13m
userprofile-xxxxxxxxxx-xxxxx                2/2     Running        0          22h
```

<br>

## Traffic Routing

Let's start with a [Canary Release][1] of the new version of the user profile service.  You'll route 90% of user traffic to version 1 and route 10% of traffic to the latest version.

<blockquote>
<i class="fa fa-terminal"></i>
View the virtual service in your favorite editor or via bash:
</blockquote>

```execute
cat ./config/istio/virtual-service-userprofile-90-10.yaml
```

Output (snippet):
```
...
---
  http:
  - route:
    - destination:
        host: userprofile
        subset: v1
      weight: 90
    - destination:
        host: userprofile
        subset: v3
      weight: 10 
---
...
```

The weights determine the amount of traffic sent to the service subset.

<blockquote>
<i class="fa fa-terminal"></i>
Deploy the routing rule:
</blockquote>

```execute
oc apply -f ./config/istio/virtual-service-userprofile-90-10.yaml
```

<blockquote>
<i class="fa fa-terminal"></i>
If you aren't already (from the Grafana lab) - send load continuously to the user profile service:
</blockquote>

```execute
while true; do curl -k -s -o /dev/null $GATEWAY_URL/profile; done
```

<br>

Inspect the change in Kiali.
<blockquote>
<i class="fa fa-desktop"></i>
Navigate to 'Graph' in the left navigation bar. 
</blockquote>

<p><i class="fa fa-info-circle"></i> If you lost the URL, you can retrieve it via:</p>

`echo $KIALI_CONSOLE`

<blockquote>
<i class="fa fa-desktop"></i>
Switch to the 'Versioned app graph' view and change to 'Last 1m'.  
</blockquote>
<blockquote>
<i class="fa fa-desktop"></i>
Change the 'No edge labels' dropdown to 'Request Distribution'.  
</blockquote>

The traffic splits between versions 1 and 3 of the user profile service at roughly 90% and 10% split.

<img src="images/kiali-userprofile-90-10.png" width="1024"><br/>
*Kiali Graph with 90-10 Traffic Split*

By doing this, you can isolate the new user profile experience for a small subset of your users without impacting everyone at once.  

Once you are comfortable with the change, you can increase the traffic load to the latest version.


<blockquote>
<i class="fa fa-terminal"></i>
View the virtual service in your favorite editor or via bash:
</blockquote>

```execute
cat ./config/istio/virtual-service-userprofile-50-50.yaml
```

Output (snippet):
```
...
---
  http:
  - route:
    - destination:
        host: userprofile
        subset: v1
      weight: 50
    - destination:
        host: userprofile
        subset: v3
      weight: 50
---
...
```

In this example, you will route traffic evenly between the two versions. This is a technique that could be used for advanced deployments, for example A/B testing.

<blockquote>
<i class="fa fa-terminal"></i>
Deploy the routing rule:
</blockquote>

```execute
oc apply -f ./config/istio/virtual-service-userprofile-50-50.yaml
```

<blockquote>
<i class="fa fa-terminal"></i>
If you aren't already - send load to the user profile service:
</blockquote>

```execute
while true; do curl -k -s -o /dev/null $GATEWAY_URL/profile; done
```

<br>

Inspect the change again in Kiali.

<blockquote>
<i class="fa fa-desktop"></i>
Navigate to 'Graph' in the left navigation bar.
</blockquote>
<blockquote>
<i class="fa fa-desktop"></i>
Switch to the 'Versioned app graph' view.  Change the 'No edge labels' dropdown to 'Request Distribution'.  
</blockquote>

You should see a roughly 50/50 percentage split between versions 1 and 3 of the user profile service.

<img src="images/kiali-userprofile-50-50.png" width="1024"><br/>
*Kiali Graph with 50-50 Traffic Split*

Finally, you are ready to roll this new version to everyone.

<br>

<blockquote>
<i class="fa fa-terminal"></i>
View the virtual service in your favorite editor or via bash:
</blockquote>

```execute
cat ./config/istio/virtual-service-userprofile-v3.yaml
```

Output (snippet):
```
...
---
  http:
  - route:
    - destination:
        host: userprofile
        subset: v3
---
...
```

<blockquote>
<i class="fa fa-terminal"></i>
Deploy the routing rule:
</blockquote>

```execute
oc apply -f ./config/istio/virtual-service-userprofile-v3.yaml
```

<blockquote>
<i class="fa fa-terminal"></i>
If you aren't already - Send load to the user profile service:
</blockquote>

```execute
while true; do curl -k -s -o /dev/null $GATEWAY_URL/profile; done
```

<br>

Inspect the change again in Kiali.
<blockquote>
<i class="fa fa-desktop"></i>
Navigate to 'Graph' in the left navigation bar. 
</blockquote>
<blockquote>
<i class="fa fa-desktop"></i>
Switch to the 'Versioned app graph' view.  Change the 'No edge labels' dropdown to 'Request Distribution'.  
</blockquote>


You should see traffic routed to v3 of the user profile service.

<img src="images/kiali-userprofile-v3.png" width="1024"><br/>
*Kiali Graph with v3 Routing*

<br>

Let's test this version of the profile service in the browser.

<blockquote>
<i class="fa fa-desktop"></i>
Navigate to the 'Profile' section in the header.  
</blockquote>

<p><i class="fa fa-info-circle"></i> If you lost the URL, you can retrieve it via:</p>

```execute
echo $GATEWAY_URL
```

<br>

You should see the following:

<img src="images/app-profilepage-v3.png" width="1024"><br/>
 *Profile Page*

<br>

## Summary

Congratulations, you configured traffic splitting in Istio!

A few key highlights are:

* We can change the percentage of traffic sent to different versions of services by modifying the 'weight' parameter in a Virtual Service
* The Kiali service graph captures traffic splitting dynamically as traffic flows in the mesh

[1]: https://martinfowler.com/bliki/CanaryRelease.html
[2]: https://martinfowler.com/bliki/BlueGreenDeployment.html
