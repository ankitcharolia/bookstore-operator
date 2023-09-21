# BookStore Operator
A Helm based BookStore Operator

Helm based operators are quite easy to build and are preferred to deploy a **stateless application** using operator pattern.

### How to generate boilerplate code for Helm based operator and build bookstore operator
```bash
# initialize the project
operator-sdk init --domain charolia.io --plugins helm

# Create a simple bookstore API using Helm’s built-in chart boilerplate
operator-sdk create api --version v1alpha1 --kind BookStore
```


**Configure the operator’s image registry**
```bash
+IMAGE_TAG_BASE ?= ankitcharolia/bookstore-operator
-IMG ?= controller:latest
+IMG ?= $(IMAGE_TAG_BASE):$(VERSION)
```

**Build and push BookStore operator’s image**
```bash
make docker-build docker-push
```

### Option-1
```bash
# Apply CRDs to the kube cluster
kubectl apply -k config/crd/
```
```bash
# Apply bookstore operator manifests to the kube cluster
kubectl apply -f deploy/manifests/bookstore-operator.yaml
kubectl apply -f deploy/manifests/bookstore-cr.yaml 
```

### Option-2
**Make sure to login to redhat registry**
```bash
docker login registry.redhat.io
Username: ankitcharolia
Password: ******
Login Succeeded
```

**Create a kubernetes secret for docker config json**
```bash
$ kubectl create secret generic regcred --from-file=.dockerconfigjson=/home/acharolia/.docker/config.json --type=kubernetes.io/dockerconfigjson (-n bookstore-operator-system)
secret/regcred created
```

**NOTE:** Use docker config json as a secret to pull the image from redhat registry
```bash
spec:
  template:
    spec:
      imagePullSecrets:
      - name: regcred
```
### Run the Operator
```bash
make deploy
```
**NOTE:** This deploys the CRDs to the Kubernetes Cluster.

```bash
$ kubectl get crd | grep bookstore
bookstore.charolia.io                            2023-09-13T19:50:54Z
```

**Create a Bookstore CR**
```bash
kubectl apply -f config/samples/bookstore_v1alpha1.yaml
```

### Clean up the resources:
```bash
kubectl delete -f config/samples/bookstore_v1alpha1.yaml
```

**Note:** Make sure the above custom resource has been deleted before proceeding to run make undeploy, as helm-operator’s controller adds finalizers to the custom resources. Otherwise your cluster may have dangling custom resource objects that cannot be deleted.

```bash
make undeploy
```

### Reference
[How-To Example](https://help.ovhcloud.com/csm/en-public-cloud-kubernetes-deploy-helm-operator?id=kb_article_view&sysparm_article=KB0049804#customresources)