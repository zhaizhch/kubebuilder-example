# How to create a multi-version crd
It is a learn example from website, the most content is the same as official website, but fix some error that I met when learn from official website
## scaffolding out our project
```sh
# create a project directory, and then run the init command.
mkdir project
cd project
# we'll use a domain of tutorial.kubebuilder.io,
# so all API groups will be <group>.tutorial.kubebuilder.io.
# before init it, you should make sure upgrade kubebuilder to the latest version(support go/v4)
kubebuilder init --domain tutorial.kubebuilder.io --repo tutorial.kubebuilder.io/project --plugins=go/v4
```
## create api and controller
```sh
kubebuilder create api --group batch --version v1 --kind CronJob
```
press y for “Create Resource” and “Create Controller”

## design the api and controller depending on your business scenario
you can learn from official website(https://book.kubebuilder.io/, recommend) or Chinese website(https://cloudnative.to/kubebuilder/)(it is not the latest version) or our code
```attention
you should add RBAC markers for you have the right permission when you run it
// +kubebuilder:rbac:groups=batch.tutorial.kubebuilder.io,resources=cronjobs,verbs=get;list;watch;create;update;patch;delete
// +kubebuilder:rbac:groups=batch.tutorial.kubebuilder.io,resources=cronjobs/status,verbs=get;update;patch
```
## create webhook
```sh
kubebuilder create webhook --group batch --version v1 --kind CronJob --defaulting --programmatic-validation
```
## deploy cert-manager
```sh
kubectl apply -f https://github.com/cert-manager/cert-manager/releases/download/v1.12.0/cert-manager.yaml
# you should create secret used for controller
kubectl apply -f config/certmanager/certificate.yaml
```
## running and deploy the controller
```sh
# generate the manifests, modify the Makefile manifests to 
.PHONY: manifests
manifests: controller-gen ## Generate WebhookConfiguration, ClusterRole and CustomResourceDefinition objects.
	$(CONTROLLER_GEN) rbac:roleName=manager-role crd:maxDescLen=0 webhook paths="./..." output:crd:artifacts:config=config/crd/bases
#avoid the error that length too long
make manifests
# install crd
make install
#run controller
export ENABLE_WEBHOOKS=false #used for close the webhook, if you create multi-version crd, it must be true
make run
# build controller image, you can set the image that you can push to your harbor
# modify the Dockerfile 

#RUN GO111MODULE=on GOPROXY=https://mirrors.aliyun.com/goproxy/ CGO_ENABLED=0 GOOS=${TARGETOS:-linux} GOARCH=${TARGETARCH} go build -a #-o manager cmd/main.go

make docker-build docker-push IMG=<some-registry>/<project-name>:tag
make deploy IMG=<some-registry>/<project-name>:tag
```
## create the second version of crd
```sh
kubebuilder create api --group batch --version v2 --kind CronJob
```
Press y for “Create Resource” and n for “Create Controller”
## conversion
you can learn how to convert between diffrent version of crd form official website(https://book.kubebuilder.io/multiversion-tutorial/conversion-concepts.html).
you should create file project/api/v1/*_conversion.go and project/apu/v2/*_conversion.go.
you should implement the hub and spokes in the file created by the previous step

you should create webhook of v2
```sh
kubebuilder create webhook --group batch --version v1 --kind CronJob --conversion
```
## Testing
before you deploy, you should 
```sh
Enable patches/webhook_in_<kind>.yaml and patches/cainjection_in_<kind>.yaml in config/crd/kustomization.yaml file.

Enable ../certmanager and ../webhook directories under the bases section in config/default/kustomization.yaml file.

Enable manager_webhook_patch.yaml and webhookcainjection_patch.yaml under the patches section in config/default/kustomization.yaml file.

Enable all the vars under the CERTMANAGER section in config/default/kustomization.yaml file.
```
in config/default/kustomization.yaml
1. change the namespace name to system
2. comment the namePrefix: crd-
then, you can deploy the resource in system namespace, or you will deploy them in crd-system, when you test, you may meet some error occured by it.






# crd
// TODO(user): Add simple overview of use/purpose

## Description
// TODO(user): An in-depth paragraph about your project and overview of use

## Getting Started
You’ll need a Kubernetes cluster to run against. You can use [KIND](https://sigs.k8s.io/kind) to get a local cluster for testing, or run against a remote cluster.
**Note:** Your controller will automatically use the current context in your kubeconfig file (i.e. whatever cluster `kubectl cluster-info` shows).

### Running on the cluster
1. Install Instances of Custom Resources:

```sh
kubectl apply -f config/samples/
```

2. Build and push your image to the location specified by `IMG`:

```sh
make docker-build docker-push IMG=<some-registry>/crd:tag
```

3. Deploy the controller to the cluster with the image specified by `IMG`:

```sh
make deploy IMG=<some-registry>/crd:tag
```

### Uninstall CRDs
To delete the CRDs from the cluster:

```sh
make uninstall
```

### Undeploy controller
UnDeploy the controller from the cluster:

```sh
make undeploy
```

## Contributing
// TODO(user): Add detailed information on how you would like others to contribute to this project

### How it works
This project aims to follow the Kubernetes [Operator pattern](https://kubernetes.io/docs/concepts/extend-kubernetes/operator/).

It uses [Controllers](https://kubernetes.io/docs/concepts/architecture/controller/),
which provide a reconcile function responsible for synchronizing resources until the desired state is reached on the cluster.

### Test It Out
1. Install the CRDs into the cluster:

```sh
make install
```

2. Run your controller (this will run in the foreground, so switch to a new terminal if you want to leave it running):

```sh
make run
```

**NOTE:** You can also run this in one step by running: `make install run`

### Modifying the API definitions
If you are editing the API definitions, generate the manifests such as CRs or CRDs using:

```sh
make manifests
```

**NOTE:** Run `make --help` for more information on all potential `make` targets

More information can be found via the [Kubebuilder Documentation](https://book.kubebuilder.io/introduction.html)

## License

Copyright 2023.

Licensed under the Apache License, Version 2.0 (the "License");
you may not use this file except in compliance with the License.
You may obtain a copy of the License at

    http://www.apache.org/licenses/LICENSE-2.0

Unless required by applicable law or agreed to in writing, software
distributed under the License is distributed on an "AS IS" BASIS,
WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
See the License for the specific language governing permissions and
limitations under the License.

