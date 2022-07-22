<h1> Cluster Hardening 15%</h1>

<h2>Restrict access to Kubernetes API</h2>
<p>fill with info</p>

<li>fill with info</li>
<li>fill with info</li>

<h3> fill with question</h3>

<details><summary>Answer</summary>

```bash
#replace with answers
```

</details>


<h2>Use Role Based Access Controls to minimize exposure</h2>

<h3> Role Based Access Control (RBAC)</h3>
<li>a method of regulating access to computer or network resources based on the roles of individual users within your organization."</li>
<li>fill with info. This will help you retrict access to kubernetes resources when accessed by Users or ServiceAccounts</li>
<li>Works with Roles and Bindings</li>
<li>Specify what is Allowed, everything else is denied</li>

<h3> Resources broken into Namespaced vs Non Namespaced</h3>
<p> to view which resources in K8s that are namespaced or not type these commands</p>

```bash

# print namespaced resources
kubectl api-resources --namespaced=true

# print non namespaced resources
kubectl api-resources --namespaced=false
```

<h3> Role VS ClusterRole</h3>
<p> Both Role and clusterRole set permissions. They have rules that defines which actions are being allowed against a resource (i.e get pods). However, they work at different levels. Roles are tied to a namespace. ClusterRoles can apply to the entire cluster (meaning all namespaces, all resources, all non-namespace resources). BE CAREFUL WITH CLUSTERROLES, THIS GIVES THE USER CLUSTER LEVEL POWER<p> 
<h3> fill with question</h3>

<h3> RBAC Permission are additive </h3>
<p> RBAC offers you the option to follow least privilege security model. By specifiying that is allowed, will automatically deny everything else. Permission are additive. For example if a user has been bound to multiple roles or clusterRoles, they will inherit all the permissions combined from these roles and clusterRoles.</p>

<h3> RoleBinding vs ClusterRoleBinding</h3>
<p> Once roles or clusterRoles are defined, you can create roleBinding or ClusterRoleBinding to bind the set of permissions to something (i.e users, serviceAccounts). Similarly to the bullet above, roleBinding only applies to resources in one namespace. ClusterRoleBinding can bind to cluster resources, non-namespaces resources, and all namespaces</p>

<h3> Create a role named secret-maintainer that allows a user to get, watch, list all secrets in the qa namespace</h3>
<details><summary>Answer</summary>

```bash
# imperative using kubectl command
kubectl create role secret-maintainer --verb=get,watch,list --resource=secrets --namespace qa

# declarative creating file then applying
# vi secret-maintainer.yaml
metadata:
  namespace: qa
  name: secret-maintainer
rules:
- apiGroups: [""]
  resources: ["secrets"]
  verbs: ["get", "watch", "list"]

kubectl apply -f secret-maintainer.yaml
```

</details>

<h3> Create a clusterRole named pod-reader that allows user to perform "get", "watch" and "list" on pods</h3>
<details><summary>Answer</summary>

```bash
# imperative using kubectl command
# Create a ClusterRole named "pod-reader" that allows user to perform "get", "watch" and "list" on pods
kubectl create clusterrole pod-reader --verb=get,list,watch --resource=pods

# declarative creating file then applying
# vi secret-maintainer.yaml
metadata:
  name: pod-reader
rules:
- apiGroups: [""]
  resources: ["pods"]
  verbs: ["get", "watch", "list"]

kubectl apply -f secret-maintainer.yaml
```

</details>

<h3> Scenario 1: 
<li>Create a namespace named qa</li>
<li> User James should only "get" and "list" secrets in the qa namespace</li>
</li> Verify that authentication is setup properly using the auth can-i command</li>
</h3>
<details><summary>Answer</summary>

```bash
##### imperative solution #####

# creating namespace
kubectl create namespace qa

# create role named secret-readers with verb=get,list in the developer namespace
kubectl create role secret-readers --verb=get,list --resource=secrets -n qa

# bind role secret-readers to user james
kubectl create rolebinding secret-readers --role=secret-readers --user=james

# verify that user James has the permission to get and list secrets in the qa namespace
kubectl  auth can-i get secrets --as james -n qa
kubectl auth can-i get secrets --as james -n qa

#### Declarative solution (ONLY WANTED TO SHOW A DEMO OF THIS. DURING EXAM USE IMPERATIVE INSTEAD OF DECLARATIVE. ONLY USE DECLARATIVE IF YOU MUST.) #####

#create namespace
kubectl create namespace qa # no reason to not use imperative to create a namespace

# create role by using the imperative command to generate the yaml file (which you can save, make changes to for later)
kubectl create role secret-readers --verb=get,list --resource=secrets -n qa -o yaml --dry-run=client > role-secret-readers.yaml

# cat secret-readers.yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  creationTimestamp: null
  name: secret-readers
  namespace: qa
rules:
- apiGroups:
  - ""
  resources:
  - secrets
  verbs:
  - get
  - list

kubectl apply -f role-secret-readers.yaml

# bind role secret-readers to user james (using the imperative approach, we can generate the declarative yaml file that can be modified and stored for later use)
kubectl create rolebinding secret-readers -n qa --role=secret-readers --user=james -oyaml --dry-run=client > rolebinding-secret-readers.yaml

# cat rolebinding-secret-readers.yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: RoleBinding
metadata:
  creationTimestamp: null
  name: secret-readers
  namespace: qa
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: Role
  name: secret-readers
subjects:
- apiGroup: rbac.authorization.k8s.io
  kind: User
  name: james

kubectl apply -f rolebinding-secret-readers.yaml

```

</details>

<h3> Scenario 2: 
<li>Create a ClusterRole named deployment-killer that will allow users to delete deployments</li>
<li> User James should be allowed to delete deployments in all namespaces</li>
<li> User Tim can only delete deployments in the developer namespace</li>
<li> Using the auth can-i command, verify that the permissions are set properly</li>
</h3>
<details><summary>Answer</summary>

```bash
# create clusterRole named deployment-killer with verb=delete applied to deployments resources. 
kubectl create clusterrole deployment-killer --verb delete --resource=deployments

# bind clusterRole deployment-killer to user james. 
# James should be able to delete deployments in all namespace
kubectl create clusterrolebinding cluster-deployment-killer --clusterrole deployment-killer --user=james

# bind clusterRole deployment-killer to user Tim. 
# remember that user Tim should only be allowed to delete deployments in the developer namespace
kubectl create rolebinding developer-deployment-killer --clusterrole deployment-killer --user=tim 
# verify that James can delete deployments in any namespaces 
kubectl  auth can-i delete deploy --as james -n qa
kubectl auth can-i delete deploy --as james -n developer

# verify that Tim can only delete deployments in the developer namespace
kubectl  auth can-i delete deploy --as tim -n qa
kubectl auth can-i delete deploy --as tim -n developer

```

</details>



<h2>Exercise caution in using service accounts e.g. disable defaults, minimize permissions on newly created ones </h2>
<p>fill with info</p>

<li>fill with info</li>
<li>fill with info</li>

<h3> fill with question</h3>

<details><summary>Answer</summary>

```bash
#replace with answers
```

</details>


<h2>Update Kubernetes frequently</h2>

<p>fill with info</p>

<li>fill with info</li>
<li>fill with info</li>

<h3> fill with question</h3>

<details><summary>Answer</summary>

```bash
#replace with answers
```

</details>