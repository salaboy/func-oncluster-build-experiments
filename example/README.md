# OnCluster Build for a `func` project
This example runs a pipeline that builds and deploy a `func` project which the source code lives in github. 

To accomplish this, we will install a `func` as two Tekton Tasks and we will use them inside a Tekton Pipeline that will: 
- Clone Repository from Github
- Use `func build` to build the container using Buildpacks
- Use `func deploy` to deploy the `func` project to the Cluster. 

Another approach would be to use the Buildpacks Tekton Task and have a pipeline that do: 
- Clone Repository from Github
- Use buildpack task to build the function project
- Do some manipulation of the `func.yaml` file for deployment
- Use `func deploy` to deploy the function in the cluster

![Pipelines](func-on-cluster-pipeline.png)

## Notes

- It will be great if `func build` and `func deploy` can be parameterized to not need the `func.yaml` file, so for example the pipeline can just use the SHA of the image to do the deploy
  - Also this avoids the commands writing back info in the `func.yaml` file
- The buildpacks pipeline is still WIP, not sure if it is worth it
- It will be interesting to define remote build and remote deploy.. so you can choose what to do where. 


## Pre Requistes
- Kubernetes Cluster
- Tekton installed in the cluster
- Knative Serving installed in the cluster


## Installing `func` Tekton resources

This repository define two Custom Tekton Tasks `func build` and `func deploy` to install them you can run:  

```
kubectl apply --recursive -f catalog/task/

```

These two tasks are using a custom version of a Docker image that I am hosting in Docker Hub under my user `salaboy` which fixes some bugs in `func`

```salaboy/func-2e37ecdd2ee11985d861179f5d0a0fbb@sha256:33468313582069d2e6ea850cd526858918db215c1bf98558ece2f8967201937f
```
I've built this Docker image by running:
```
`ko publish ./cmd/func`
```


`func` will need to have your docker registry credentials to be able to push the produces container image to a public registry. 


```
kubectl create secret docker-registry regcred --docker-server=https://index.docker.io/v1/ --docker-username=DOCKER_USERNAME --docker-password=DOCKER_PASSWORD --docker-email DOCKER_EMAIL
```

You can now install the example pipeline with and a required PVC for sources and cache:


```
kubectl apply -f examples/pvc.yaml
kubectl apply -f examples/func-onclusterbuild-build-deploy-pipeline.yaml

```

You can now create new instances of this pipeline by running: 

```
kubectl apply -f examples/pr.yaml
```

**Note**: I am using a `PipelineRun` because I need to set up a ServiceAccount for Build and another one for Deploy, and I haven't found the way to do it using `tkn`

```
tkn pipeline start func-pipeline -s dockerconfig -w name=sources,claimName=source-pvc,subPath=source -w name=cache,claimName=source-pvc,subPath=cache -w name=dockerconfig,secret=regcred

```

