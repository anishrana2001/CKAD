# **CANARY Deployment**

#
-  _**Will create 2 Deployments**_ 
- _**attach the same label to both deployments**_
- _**create 1 service and point to both deployments with the help of "same" label.**_
---
---
### Task 1:
### - Create a namespace called `tiger` namespace.
### - Create a deployment called `blue` with `5 replicas`, using the nginx image `1.26.3` inside the `tiger` namespace.
### - Expose `port 80` for the nginx containers.
### Task 2:
### - Create a Service called `web-srv` to route traffic to `blue`.
### Task 3:
### - Create a new deployment called `green` , with `5 replicas`, using the nginx image `1.26.4` inside the namespace.
### - Expose `port 80` for the nginx containers.
### Task 4:
### Now, use the Service called `web-srv` to route traffic to `blue` & `green` deployments.

### Task 5
### Split traffic between `blue=70%` and `green=30%`   👈👈👈

 💡💡💡 _**Add the common label to both deployments.**_

 <details>
<summary>> Solution</summary>


 <details>
<summary> Task 1: </summary>
## Create a namespace called `tiger` namespace.
This is the content of the collapsible section. You can include any Markdown-formatted text, lists, or code here.

</details>
</details>
