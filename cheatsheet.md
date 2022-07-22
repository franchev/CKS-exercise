<h1> Exam Cheatsheets </h1>

<p> During the exam, it is best to use imperative kubectl commands to quickly create resources. You can use the kubectl --help to view a list of commands that are supported. You can also use the dry-run option to generate yaml files that can then be edited to add additional parameters. You only have two hours to answer 15-20 tasks based questions. therefore, the quicker you can create, modify your resources, the better. Below, you'll find a list of commands that will help you be faster when working in the kubernetes cluster</p>

#impersonate service account to see if it can list pods
kubectl auth can-i list pods --as=system:serviceaccount:default:mysvcaccount1

# impersonate service account see if it can get pods
kubectl get pods -v 6 --as=system:serviceaccount:default:mysvcaccount1

# impersonate user james to see if it can list secrets
kubectl auth can-i get secrets --as james -n qa
