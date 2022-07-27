<h1> Cluster Hardening 15%</h1>

<h2>Restrict access to Kubernetes API</h2>

<h3>List of external links for this domain</h3>

<li>https://kubernetes.io/docs/concepts/security/controlling-access/</li>

<p>Kubernetes API is the one of the most important aspect of kubernetes. All requests are api requests which goes through this workflow:</p>
<li>Authentication: verifies who you are (either a user, serviceAccount, or coming annonymously)</li>
<li>Authorization: verifies if the user or SA has access to perform the action that they are requesting</li>
<li>Admission Control: validates limits that are set in the cluster (i.e users are only allowed to create up to 100 pods). Admission control can be defined by the kubernetes administrator. </li>

<p> When securing your kubernetes cluster, you should think of a list of restrictions to apply. For example:</p>
<li>Do not allow annonymous requests</li>
<li>Close ports that are deemed to be insecure</li>
<li>Don't expose APIServer outside of your network</li>
<li>Restrict access from nodes to API(NodeRestriction)</li>
<li>Manage access through RBAC (follow least privilege model)</li>
<li>Prevent pods from accessing the API</li>

<h3> Anonymous Access</h3>

<p> You can disable anonymous requests in your k8s cluster by running. This would make your cluster more secure. You can do that by login to your kubernetes master node and follow steps below<p>

```bash

# first connect to your master node

# check if you can connect the api anonymously
curl https://localhost:6443 -k

# under the command section add: - --anonymous-auth=false
vim /etc/kubernetes/manifests/kube-apiserver.yaml

# wait for changes to apply. Making this change will create a new api pod
kubectl -n kube-system get pod | grep api

# rerun curl command to verify
curl https://localhost:6443 -k

# the api server actually needs to make anonymous calls for its livenessProbe check, therefore we need to re-enable anonymous-auth. under command block, remove: - --anonymous-auth=false
vim /etc/kubernetes/manifests/kube-apiserver.yaml

```

<h3> NodeRestriction AdmissionController</h3>

<p>This admission controller limits the Node and Pod objects a kubelet can modify. In order to be limited by this admission controller, kubelets must use credentials in the system:nodes group, with a username in the form system:node:<nodeName>. Such kubelets will only be allowed to modify their own Node API object, and only modify Pod API objects that are bound to their node. kubelets are not allowed to update or remove taints from their Node API object.

The NodeRestriction admission plugin prevents kubelets from deleting their Node API object, and enforces kubelet modification of labels under the kubernetes.io/ or k8s.io/ prefixes as follows:

Prevents kubelets from adding/removing/updating labels with a node-restriction.kubernetes.io/ prefix. This label prefix is reserved for administrators to label their Node objects for workload isolation purposes, and kubelets will not be allowed to modify labels with that prefix.
Allows kubelets to add/remove/update these labels and label prefixes:
kubernetes.io/hostname
kubernetes.io/arch
kubernetes.io/os
beta.kubernetes.io/instance-type
node.kubernetes.io/instance-type
failure-domain.beta.kubernetes.io/region (deprecated)
failure-domain.beta.kubernetes.io/zone (deprecated)
topology.kubernetes.io/region
topology.kubernetes.io/zone
kubelet.kubernetes.io/-prefixed labels
node.kubernetes.io/-prefixed labels
Use of any other labels under the kubernetes.io or k8s.io prefixes by kubelets is reserved, and may be disallowed or allowed by the NodeRestriction admission plugin in the future.

Future versions may add additional restrictions to ensure kubelets have the minimal set of permissions required to operate correctly.

</p>



</details>


<h2>Use Role Based Access Controls to minimize exposure</h2>

<h3>Link to documentation on RBAC</h3>

<li>https://kubernetes.io/docs/reference/access-authn-authz/rbac/</li>

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

<h3> ServiceAccounts(SA) revisited</h3>
<p>We touched briefly on serviceAccount in the RBAC section above. Now, we'll focus on it a bit more for this section. A few things to know about serviceAccounts :</p>

<li>they are namespaced</li>
<li>every namespace has a default ServiceAccount used by pods</li>
<li>Can be used to talk to k8s api</li>

<p>When you create a ServiceAccount(SA), it'll automatically create a kubernetes secrets that will hold the token that used by the SA to make api calls within the k8s cluster</p>


<h4> Create a ServiceAccount named connector in the default namespace. Then create a pod using image nginx with that same name to use that serviceAccount</h4>

<details><summary>Answer</summary>

```bash
#create serviceAccount connector
kubectl create sa connector

# create pod connector using image ngnix. We have to do one small modification to the pod, so we'll generate the pod.yaml file using imperative command. We'll send that to a file. We'll modify the file to have the pod point to the connector SA, then we'll apply to the k8s cluster
kubectl run connector --image=nginx -oyaml --dry-run=client > connector-pod.yaml
vim connector-pod.yaml 
apiVersion: v1
kind: Pod
metadata:
  creationTimestamp: null
  labels:
    run: connector
  name: connector
spec:
  serviceAccountName: connector
  containers:
  - image: nginx
    name: connector
    resources: {}
  dnsPolicy: ClusterFirst
  restartPolicy: Always
status: {} 

kubectl apply -f connector-pod.yaml

```

</details>

<h3> Limit ServiceAccount using RBAC</h3>

<p> by default, all pods use the default serviceAccount defined in their namespace. By default, this serviceAccount(default) doesn't have any permissions. With that said, if someone were to modify that serviceAccount(default) and attached some permissions to it, then all pods in that namespace would inherrit those permissions</p>

<p> Using RBAC, you can bind serviceAccount to roles or clusterroles. Here's a quick example<p>

<h4>Bind the connector serviceAccount to the edit clusterrole (this is a clusterrole that exists by default). Check if connector ServiceAccount can delete secrets using the auth can-i command</h4>

```bash
# check if connector serviceAccount can delete secrets
kubectl auth can-i delete secrets --as system:serviceaccount:default:connector

# create clusterrolebinding named connector and bind serviceAccount connector to clusterrole edit
kubectl create clusterrolebinding connector --clusterrole edit --serviceaccount default:connector

# now again verify if connector serviceAccount can delete secrets
kubectl auth can-i delete secrets --as system:serviceaccount:default:connector
```

<h3>Limit mounting serviceAccount token in a pod</h3>

<p> Following the least privilege concept, you shouldn't give a pod access that they don't need. Therefore, you can associate disable the mounting of a serviceAccount token on a pod. This is a very simple change. Here's a quick example doing that</p>

```bash
# let's create a service account name developer in the default namespace
kubectl create sa developer 

# let's create the definition of a pod named nginx with image nginx.
# then we'll update the pod to use the developer sa, and we'll also disable the mounting on that sa token
kubectl run nginx --image=nginx --dry-run=client -o yaml > nginx.yaml

vim nginx.yaml
apiVersion: v1
kind: Pod
metadata:
  creationTimestamp: null
  labels:
    run: nginx
  name: nginx
spec:
  serviceAccountName: developer
  automountServiceAccountToken: false
  containers:
  - image: nginx
    name: nginx
    resources: {}
  dnsPolicy: ClusterFirst
  restartPolicy: Always
status: {}

kubectl apply nginx.yaml
```

<h4> Scenario 1
<li>create a pod named test1 with image nginx in the default namespace</li>
<li>run command against the pod, check if the default serviceAccount is mounted on the pod</li> 
<li>Modify the default service account, disable auto mount of the token</li>
<li>Reconnect to test1 pod, and check if secret associated with the serviceAccount is still mounted</li></h4>
<details><summary>Answer</summary>

```bash

# create pod named test1 with image nginx in default namespace 
kubectl run test1 --image=nginx 

# run command against the pod, and check if serviceAccount secret is mounted
k exec test1 -- mount | grep secret
tmpfs on /run/secrets/kubernetes.io/serviceaccount type tmpfs (ro,relatime)

# let's edit the default service account and disable auto mount of token. Add automountServiceAccountToken: 
kubectl edit sa default

apiVersion: v1
kind: ServiceAccount
automountServiceAccountToken: false
...

# run command against the pod, and check if serviceAccount secret is mounted
kubectl exec test1 -- mount | grep secret
kubectl exec test1 -- cat: /var/run/secrets/kubernetes.io/serviceaccount/token

```

</details>

<h2>Update Kubernetes frequently</h2>

<h3> Upgrade your kubernetes cluster</h2>

<p>It's recommended to upgrade your k8s cluster often. Let's discuss how to upgrade your cluster</p>

<li> Upgrade master components: apiserver, controller-manager, scheduler</li>
<li> Upgrade worker components: kubelet, kube-proxy</li>
<li> Components should use the same minor version as api server (You can have one minor version below, but recommendation is to have them all at the same minor version)</li>

<p> Steps for upgrading your nodes <p>
<li>kubectl drain: safely evict all pods from the nodes and to mark node as SchedulingDisabled (you can also use kubectl cordon to mark a node as unschedulable)</li> 
<li>After all pods have been evicted, then do the upgrades on the node</li>
<li>Lastly, run kubectl uncordon to remark the node as schedulable (SchedulingDisabled reverted)</li>

<h3> Upgrade our cluster (control plane & nodes)</h3>

<p> Since you can use the kubernetes.io doc page, I would advise to use this link below to upgrade your kubeadm cluster. I would practice this.</p>
<li>https://kubernetes.io/docs/tasks/administer-cluster/kubeadm/kubeadm-upgrade/</li>


<h3>Upgrade your kubernetes cluster to use version 1.22</h3>

<details><summary>Answer</summary>

```bash
#follow steps in this link: https://kubernetes.io/docs/tasks/administer-cluster/kubeadm/kubeadm-upgrade/
```

</details>