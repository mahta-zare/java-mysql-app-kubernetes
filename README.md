We have a java application that uses MySQL as its database. To make this application more reliable and highly available, we use Kubernetes to orchestrate the containers of this application. 

Here are the steps to configure and deploy this application using k8s. The later steps cover the creating a Helm chart for this app.
We include deployment of this application in both Minikube and LKE.

step 1

```
helm install -f chart-values-minikube.yaml my-release oci://registry-1.docker.io/bitnamicharts/mysql

kubectl get pods
```

step 2

After building the jar file of my application using gradlew and building the docker image, I login to docker and push it to my private repository:

```
docker login
docker push mahtazare/test:java-mysql-project-1.2-SNAPSHOT
```

To pull the image from the private repository on Docker Hub, we create a 'my-registry-key' secret:

```
kubectl create secret docker-registry my-registry-key \
--docker-server=docker.io \
--docker-username=mahtazare \
--docker-password=<my-docker-hub-pass>
```

Now, we need to create the k8s configuration files for each of the components involved.

```
kubectl apply -f db-config.yaml
kubectl apply -f db-secret.yaml
kubectl apply -f java-mysql-app.yaml

kubectl get pods -l app=java-mysql-app
```

To access the application, create a post-forwarding and access the running application on localhost:8080
```
kubectl post-forward service/java-mysql-app-service 8080:8080
```


step 3
In this step, we deploy phpmyadmin to access MySQL UI. To do this, a deployment and and its corresponding service are created.

```
kubectl apply -f phpmyadmin.yaml

kubectl get pods -l app=phpmyadmin
```

step 4
To publish the application for users, we install an nginx ingress controller and create an ingress component.

```
# enable ingress addons
minkube addons enable ingress
```

Create ingress component and apply:
```
kubectl apply -f java-mysql-app-ingress.yaml
```



In this section, we create Helm Charts of for the java application.
Next, we create a values file as an example, and deploy the application with helmfile.

Create a Helm Chart folder structure:

```
helm create java-mysql-app
```

To validate the correctness of the created Helm Charts do:
```
helm install -f helm/values.yaml java-mysql-app helm/java-mysql-app-chart --dry-run --debug
```

If everything worked correctly, you can create a chart release with:

```
helm install -f helm/values.yaml java-mysql-app helm/java-mysql-app-chart 
```

To uninstall:
```
helm uninstall java-mysql-app
```

To uninstall with Helmfile
```
cd helm
helmfile sync

helmfile destroy
```

To host the helm chart in its own repository:
```
helm package java-mysql-app-chart
helm push java-mysql-app-chart-0.1.0.tgz <registry>
```

