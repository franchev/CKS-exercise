<h1> Cluster Setup 10%</h1>

<h2>Use Network security policies to restrict cluster level access</h2>

<h3> Network policies doc link </h3>

<li>https://kubernetes.io/docs/concepts/services-networking/network-policies/ </li>

<h3> Purpose of network policies </h3>

<p>Let's start by discussion what network policies are in k8s cluster. Then we'll discuss how they can be used to secure your cluster</p>

<p> What are network Policies? </p>

<li> Firewall rules in K8s cluster </li>
<li> They are implemented by the Network Plugins CNI (Calico / Weave)</li>
<li> They are namespaced </li>
<li> They can restrict the Ingress and/or Egress for a group of Pods based on some rules and conditions</li>

<p> What if you don't define any network policies?</p>

<li> By default all pods can access each other </li>
<li> Pods are not isolated </li>

<p> What can network policies do?</p>
<p> Practically, you can group pods and define how they can be accessed on a network level. Let's say you have some sensitive pod, like a database, well you can restrict which pods can access that database pod using network policies. Just remember that if you associate network policy to a set of pods, then only the rules in that network policy will apply. Therefore, be careful and be sure that this is the action that you want to apply</p>

<h3> Below we'll provide some examples of network policies in yaml form, see if you can figure out their impacts and on which pods.</h3>

<h4> Network policy 1</h4>

```bash
cat policy1.yaml
kind: NetworkPolicy
metadata:
  name: policy1
  namespace: default
spec:
  podSelector:
    matchLabels:
      id: frontend
  policyTypes:
  - Egress
```

<details><summary>Answer</summary>

Here's the impact of this policy1:

<li>Deny all outgoing traffic from pods with label id=frontend in the default namespace</li>

</details>

<h4> Network policy 2</h4>

```bash
cat policy2.yaml
kind: NetworkPolicy
metadata:
  name: policy2
  namespace: default
spec:
  podSelector:
    matchLabels:
      id: frontend
  policyTypes:
  - Egress
  egress:
  - to:
    - namespaceSelector:
        matchLabels:
          id: backends
    ports:
    - protocol: TCP
      port: 3306
  
  - to:
    - podSelector:
        matchLabels:
          id: backend 
```

<details><summary>Answer</summary>

Here's the impact of policy2. Understand that there are actually two rules in this policy. Either rule one or rule two will apply.

<li>This applies to all pods with label id: frontend in the default namespace</li>
<li> This is an egress rule (outbound traffic) from the pods</li>
<li> Rule1 states: that egress (outbound traffic) is allowed to namespace with label id=backends AND port 3306. Therefore, this will send traffic to any pods in the namespace backend that can receive traffic on port 3306</li>
<li> Rule2 states: that egress is allowed within the same default namespace to pods with label id: backend</li>

</details>

<h3> Now let's do some scenarios. Get some practice on </h3>
 Remember, you can use this link: https://kubernetes.io/docs/home/ and do some research during the exam. There are some areas that are excluded, so follow the documentation on the exam.

<h4> Scenario 1
<li>Create two pods named frontend & backend using the nginx image</li>
<li> expose both pods behind their respective services</li>
<li> create a deny-all policy that will prevent all ingress AND egress traffic for all pods within the default namespace</li>
<li>Using the respective services, verify that pods cannot be accessed</li>
</h4>

<details><summary>Answer</summary>

```bash
#create frontend pod
kubectl run frontend --image=nginx

# create backend pod
kubectl run backend --image=nginx

# expose frontend pod
kubectl expose pod frontend --port 80

# export backend pod
kubectl expose pod backend --port 80

# verify that you can access the service between both pods
kubectl exec frontend -- curl backend

kubectl exec backend -- curl frontend 

# create deny all network policy
vi deny-all-traffic.yaml
---
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: default-deny-all
spec:
  podSelector: {}
  policyTypes:
  - Ingress
  - Egress

kubectl apply -f deny-all-traffic.yaml

# again check if you can reach services between pods
kubectl exec frontend -- curl backend

kubectl exec backend -- curl frontend 
```

</details>

<h2>Use CIS benchmark to review the security configuration of Kubernetes components (etcd, kubelet, kubedns, kubeapi)</h2>
<p>fill with info</p>

<li>fill with info</li>
<li>fill with info</li>

<h3> fill with question</h3>

<details><summary>Answer</summary>

```bash
#replace with answers
```

</details>

<h2>Properly set up Ingress objects with security control </h2>
<p>fill with info</p>

<li>fill with info</li>
<li>fill with info</li>

<h3> fill with question</h3>

<details><summary>Answer</summary>

```bash
#replace with answers
```

</details>

<h2>Protect node metadata and endpoints</h2>
<p>fill with info</p>

<li>fill with info</li>
<li>fill with info</li>

<h3> fill with question</h3>

<details><summary>Answer</summary>

```bash
#replace with answers
```

</details>

<h2>Minimize use of, and access to, GUI elements
Verify platform binaries before deploying</h2>
<p>fill with info</p>

<li>fill with info</li>
<li>fill with info</li>

<h3> fill with question</h3>

<details><summary>Answer</summary>

```bash
#replace with answers
```

</details>