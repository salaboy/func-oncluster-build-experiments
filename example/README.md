# OnCluster Build for a `func` project
This example runs a pipeline that builds and deploy a `func` project which the source code lives in github. 

To accomplish this, we will install a `func` Tekton Task and we will use it inside a Tekton Pipeline that will: 
- Clone Repository from Github
- Use `func build` to build the container using Buildpacks
- Use `func deploy` to deploy the `func` project to the Cluster. 

## PreRequisted
- Kubernetes Cluster
- Tekton installed in the cluster
- Knative Serving installed in the cluster


## Installing `func` Tekton resources

```
kubectl apply --recursive -f catalog/task/func/0.1

```

`func` will need to have your docker registry credentials to be able to push the produces container image to a public registry. 


```
kubectl create secret docker-registry regcred --docker-server=https://index.docker.io/v1/ --docker-username=DOCKER_USERNAME --docker-password=DOCKER_PASSWORD --docker-email DOCKER_EMAIL
```

You can now install the example pipeline with: 

```
kubectl apply -f examples/func-on-cluster-build-pipeline.yaml
```

You can now create new instances of this pipeline by running: 

```
tkn pipeline start func-pipeline -s dockerconfig -w name=sources,claimName=source-pvc,subPath=source -w name=cache,claimName=source-pvc,subPath=cache -w name=dockerconfig,secret=regcred

```

