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

<h3> RBAC Permission are additive </h3>
<p> RBAC offers you the option to follow least privilege security model. By specifiying that is allowed, will automatically deny everything else. Permission are additive. For example if a user has been bound to multiple roles or clusterRoles, they will inherit all the permissions combined from these roles and clusterRoles.</p>

<h3> RoleBinding vs ClusterRoleBinding</h3>
<p> Once roles or clusterRoles are defined, you can create roleBinding or ClusterRoleBinding to bind the set of permissions to something (i.e users, serviceAccounts). Similarly to the bullet above, roleBinding only applies to resources in one namespace. ClusterRoleBinding can bind to cluster resources, non-namespaces resources, and all namespaces</p>

<h4> Create a role named secret-maintainer that allows a user to get, watch, list all secrets in the qa namespace</h4>
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

<h4> Create a clusterRole named pod-reader that allows user to perform "get", "watch" and "list" on pods</h4>
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

<h4> Scenario 1: 
<li>Create a namespace named qa</li>
<li> User James should only "get" and "list" secrets in the qa namespace</li>
</li> Verify that authentication is setup properly using the auth can-i command</li>
</h4>
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

<h4> Scenario 2: 
<li>Create a ClusterRole named deployment-killer that will allow users to delete deployments</li>
<li> User James should be allowed to delete deployments in all namespaces</li>
<li> User Tim can only delete deployments in the developer namespace</li>
<li> Using the auth can-i command, verify that the permissions are set properly</li>
</h4>
<details><summary>Answer</summary>

```bash
# create clusterRole named deployment-killer with verb=delete applied to deployments resources. 
kubectl create clusterrole deployment-killer --verb delete --resource=deployments

# bind clusterRole deployment-killer to user james. 
# James should be able to delete deployments in all namespace
kubectl create clusterrolebinding cluster-deployment-killer --clusterrole deployment-killer --user=james

# bind clusterRole deployment-killer to user Tim. 
# remember that user Tim should only be allowed to delete deployments in the developer namespace
kubectl create rolebinding developer-deployment-killer --clusterrole deployment-killer --user=tim -n developer
# verify that James can delete deployments in any namespaces 
kubectl  auth can-i delete deploy --as james -A   # all namespaces
kubectl auth can-i delete deploy --as james -n developer

# verify that Tim can only delete deployments in the developer namespace
kubectl  auth can-i delete deploy --as tim -A
kubectl auth can-i delete deploy --as tim -n developer

```

</details>


<h3> serviceAccount VS User </h3>
<li> A serviceAccount is an account that can be managed by the k8s api</li>
<li> Actually, there are no k8s User resource. K8s assumes that a cluster-independent service manages normal users (i.e AWS, GCP, etc). In this case, a user is simply someone with a cert and key to access resources in the cluster</li> 

<p> serviceAccounts are easy to understand and they are actual resources in a k8s cluster. We'll have some examples on serviceAccounts. However, let us zoom in a bit on k8s externally managed users</p>

<p> As mentioned above a user is someone that has a cert and a key. A client cert must be signed by the cluster's certificate authority (CA). te username must be under common Name /CN=james.</p>
<p> Below, i've provided an example for how to generate a client cert that is signed by the cluster's CA. I will be using openssl.<p>

<h4>Create a user cert for username: james to connect to a cluster</h4>

```bash
# Create key using openssl
openssl genrsa -out james.key 2048

# create certificate signing request using openssl. Press enter for each option. Except Common Name, please set that to james
openssl req -new -key james.key -out james.csr

# base64 encode james.csr request. This is needed to create the CertificateSigningRequest resource in kubernetes
cat james.csr | base64 -w 0

# Create CertificateSigningRequest resource using kubectl 
vim james-csr.yaml
apiVersion: certificates.k8s.io/v1
kind: CertificateSigningRequest
metadata:
  name: james
spec:
  groups:
  - system:authenticated
  request: [copy base64 encoded value from step above]
  signerName: kubernetes.io/kube-apiserver-client
  usages:
  - client auth

kubectl apply -f james-csr.yaml 

# view csr resource in kubernets, it'll be in a pending state
kubectl get csr

# approve james certificate
kubectl certificate approve james

# copy certificate generated after approval
kubectl get csr james -oyaml 

# under status section grab certificate blob
# base64 decode certificate
echo "replace with certificate blob=====" | base64 -d > james.crt

# add newly created certs and keys in kubectl config file. use the --embed-certs to add actual content of key and crt files in k8s config file.
kubectl config set-credentials james --client=key=james.key --client=certificate=james.crt --embed-certs

# create context to connect the kubernetes cluster to the user james
kubectl config set-context james ==user=james --cluster=[name_of_cluster]

#Test connecting as user James by switching to the james context
kubectl config use-context james

# here you get see what kind of access james has. If user james needs more access, remember you'll need to create roles or clusterRoles and bind them to the user.
kubectl auth can-i list pods -A

```

<h4> Scenario 3: 
<li> User Andrea should be allowed to list, watch, get pods from all namespaces</li>
<li> As the administrator of the kubernetes cluster named k8s-stage, please share the certificate base64 decoded with Andrea. Also provide Andrea with a list of instruction to connect to the cluster with the certificate provided<li>
<>
</h4>
<details><summary>Answer</summary>

```bash
# create clusterRole named engineer-viewer with verb=list,watch,get applied to pods resources. 
kubectl create clusterrole engineer-viewer --verb=list,watch,get --resource=pods

# bind clusterRole engineer-viewer to user andrea. 
kubectl create clusterrolebinding engineer-viewer --clusterrole engineer-viewer --user=andrea

# verify that andrea can list all pods in all namespaces
kubectl  auth can-i get pods --as andrea -A   # all namespaces

# ping Andrea, using openssl have her generate key and csr. 
# have her share the csr file with you
openssl genrsa -out andrea.key 2048
# remember here to tell andrea to set the Common Name to be andrea
openssl req -new -key andrea.key -out andrea.csr

# base64 encode andrea.csr file
cat andrea.csr | base64 -w 0

#using base64 encoded andrea.csr, create kubernetes CertificateSigningRequest resource in the k8s-stage cluster
vim andrea-csr.yaml
apiVersion: certificates.k8s.io/v1
kind: CertificateSigningRequest
metadata:
  name: andrea
spec:
  groups:
  - system:authenticated
  request: [copy base64 encoded value from step above]
  signerName: kubernetes.io/kube-apiserver-client
  usages:
  - client auth

kubectl apply -f andrea-csr.yaml

# approve andrea's signature request
kubectl certificate approve andrea

# grab certificate to share with andrea
kubectl get csr andrea -o yaml

# base64 decode and share with Andrea
echo "base64decodeandreacert======" | base64 -d > andrea.crt

# share decoded cert with Andrea with instruction
# add newly created certs and keys in kubectl config file. use the --embed-certs to add actual content of key and crt files in k8s config file.
kubectl config set-credentials andrea --client=key=andrea.key --client=certificate=andrea.crt --embed-certs

# create context to connect the kubernetes cluster to the user andrea
kubectl config set-context andrea ==user=james --cluster=k8s-stage

#Test connecting as user James by switching to the james context
kubectl config use-context andrea

# here you get see what kind of access james has. If user james needs more access, remember you'll need to create roles or clusterRoles and bind them to the user.
kubectl auth can-i list pods -A
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