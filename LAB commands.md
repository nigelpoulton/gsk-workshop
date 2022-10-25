# LAB Commands

Run all of the following commands from the root of the repo.

## CONTAINER DEMO

Inspect the `Dockerfile` in `./latest`.

Build an image from the Dockerfile.

```
docker image build -t nigelpoulton/gskworkshop:1.0 ./latest
```

List images.

```
docker image ls
```

Push Linux AMD64 and Linux AArch64 images to Docker Hub. Be sure to substitute your own Docker Hub account.

```
docker buildx build --push \
    --platform linux/amd64,linux/arm64/v8 \
    --tag nigelpoulton/gskworkshop:1.0 \
    ./latest
```


## DEPLOYING APPS TO KUBERNETES

You must be connected to your Kubernetes cluster to run these commands.

List any existing Pods. There might be none.

```
kubectl get pods
```

Deploy the app defined in the `pod.yml` file.

```
kubectl apply -f pod.yml
```

List and describe the Pod.

```
kubectl get pods –-watch
<Snip>

kubectl describe pod hello-pod
<Snip>
```

## EXPOSING APPS ON NETWORKS

Deploy the LoadBalancer Service defined in the `svc.yml` file.

```
kubectl apply -f svc.yml
```

List the Service.

```
kubectl get svc
```

Compare Service's label selector with the label applied to the Pods.

```
kubectl describe svc gsk-lb
<Snip>

kubectl get pods --show-labels
<Snip>
```

Copy the Service's public IP and paste it into your browser on port `9000`.

 
## Cloud native superpowers – Using Deployments

Delete the Pod you previously deployed.

```
kubectl delete pod hello-pod
```             

Refresh your browser to ensure the app is no longer running.

Deploy the app from the `deploy.yml` file.

```
kubectl apply -f deploy.yml
```

List the Deployment and all Pods it is managing.

```
kubectl get deploy
<Snip>

kubectl get pods
<Snip>
```

Refresh your browser to ensure the app is up and running.

## Cloud native superpowers – Scaling

Edit the `deploy.yml` file and change the number of replicas from 5 to 2. This is in the `spec.replicas` field.

Save your change and deploy the updated app configuration.

```
kubectl apply -f deploy.yml
```

Watch the number of Pods change from 4 down to 2.

```
kubectl get pods -–watch
<Snip>

kubectl get deploy
<Snip>
```


## Cloud native superpowers – Self healing

Delete one of the Pods. You may have to run a `kubectl get pods` command to retrieve the name of a Pod.

```
kubectl delete pod <insert Pod name> --force
```

List the Pods again and notice that one of them has been running for a much shorter time than the others.

```
kubectl get pods
```

List the Pods and the K8s Node each one is running on.

```
kubectl get pods -o wide
```

Go to Linode.com and delete/recycle the any of the Nodes that is running a Pod.

Run the following command to see at least one of the Pods fail and new one(s) to be created.

```
kubectl get pods --watch
```

Edit the `deploy.yml` file again and change the number of replicas up to 10.

Save your changes and re-deploy the app again.

```
kubectl apply -f deploy.yml
```

Verify the number of Pods has increasde to 10.

```
kubectl get deploy
<Snip>

kubectl get pods
<Snip>
```

## Cloud native superpowers – Rolling updates

Refresh your browser to show the current version of the app (this will change in a later step).

Edit `deploy.yml` and change the version of the image from `1.0` to `2.0`.

Save your changes and re-deploy the app from `deploy.yml`.

```
kubectl apply -f deploy.yml
```

Monitr the Pods and the status of the rollout.

```
kubectl get pods
<Snip>

kubectl rollout status deploy hello-deploy
<Snip>
```

Refresh your browser and notice the new version of the app.


## Cloud native superpowers – Versioned rollback

Check the version history of the Deployment

```
kubectl rollout history deploy hello-deploy
```

List the two ReplicaSet objects and notice that the current one is controlling all 10 Pods and the other is controlling none.

```
kubectl get rs
```

Describe the ReplicaSet that own none of the Podsand verify the `revision`.

```
kubectl describe rs <insert old one>
```

Do the same for the ReplicaSet that owns all 10 Pods.

```
kubectl describe rs <insert new one>
```

Revert to the previous version. The version number may be different in your environment.

```
kubectl rollout undo –to-revision 2
```