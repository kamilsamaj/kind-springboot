# kind-springboot
Experimenting with Kind, SprintBoot, ServiceMesh, etc.

## Prerequisites
* asdf (see below)
* curl
* git
* docker
* jq

## Tool installation

* Get [ASDF](https://github.com/asdf-vm/asdf)
```bash
git clone https://github.com/asdf-vm/asdf.git ~/.asdf
printf 'source $HOME/.asdf/asdf.sh
source $HOME/.asdf/completions/asdf.bash
' >> $HOME/.bashrc
source $HOME/.bashrc
```

* Add and install ASDF plugins
```bash
for PLUGIN in $(cut -d ' ' -f1 << .tool-versions); do
  echo "Adding $PLUGIN"
  asdf plugin-add $PLUGIN;
done

# now install the exact plugin versions
asdf install
```

## SpringBoot application
* The application has been created by [Spring Initializr](https://start.spring.io/)
```bash
curl https://start.spring.io/starter.tgz -d dependencies=webflux,actuator | tar -xzvf -
./mvnw clean install
java -jar target/*.jar
```

* You can check the [SpringBoot Actuator](https://www.baeldung.com/spring-boot-actuators) APIs:
```bash
curl localhost:8080/actuator | jq .
```

## Containerize the application

* Build a local Docker image
```bash
./mvnw spring-boot:build-image
```

* Start the new local image
```bash
docker run -p 8080:8080 demo:0.0.1-SNAPSHOT
```

* Check the `/health` endpoint
```bash
curl localhost:8080/actuator/health
```

## Create your KIND cluster with local registry
* Start local Docker registry in your KIND k8s cluster
```bash
./run-kind-with-local-registry.sh
```

* Check your cluster is running
```bash
$ kubectl cluster-info

Kubernetes master is running at https://127.0.0.1:39029
KubeDNS is running at https://127.0.0.1:39029/api/v1/namespaces/kube-system/services/kube-dns:dns/proxy

To further debug and diagnose cluster problems, use 'kubectl cluster-info dump'.
```
```bash
$ kubectl version
Client Version: version.Info{Major:"1", Minor:"19", GitVersion:"v1.19.2", GitCommit:"f5743093fd1c663cb0cbc89748f730662345d44d", GitTreeState:"clean", BuildDate:"2020-09-16T13:41:02Z", GoVersion:"go1.15", Compiler:"gc", Platform:"linux/amd64"}
Server Version: version.Info{Major:"1", Minor:"19", GitVersion:"v1.19.1", GitCommit:"206bcadf021e76c27513500ca24182692aabd17e", GitTreeState:"clean", BuildDate:"2020-09-14T07:30:52Z", GoVersion:"go1.15", Compiler:"gc", Platform:"linux/amd64"}
```

## Push your container to your local registry
```bash
docker tag demo:0.0.1-SNAPSHOT localhost:5000/demo:0.0.1-SNAPSHOT
docker push localhost:5000/demo:0.0.1-SNAPSHOT
```

## Deploy the Application to Kubernetes
* Generate the `deployment.yaml`
```bash
kubectl create deployment demo \
    --image=localhost:5000/demo:0.0.1-SNAPSHOT \
    --dry-run=client -o=yaml > deployment.yaml

echo "---" >> deployment.yaml
kubectl create service clusterip demo \
    --tcp=8080:8080 \
    --dry-run=client -o=yaml >> deployment.yaml
```

* Apply the `deployment.yaml`
```bash
kubectl apply -f deployment.yaml
```
* Check the service is running
```bash
kubectl get all
```

* Forward your local ports to the service
```bash
kubectl port-forward svc/demo 8080:8080
```

* Check your service status
```bash
curl localhost:8080/actuator/health
```

## Clean-up
```bash
./mvnw clean
kind delete cluster
docker stop registry
docker rmi demo:0.0.1-SNAPSHOT
docker system prune
```

## Short version
If you already have everything installed, just run these steps:
```bash
asdf install
./mvnw spring-boot:build-image
./run-kind-with-local-registry.sh
docker tag demo:0.0.1-SNAPSHOT localhost:5000/demo:0.0.1-SNAPSHOT
docker push localhost:5000/demo:0.0.1-SNAPSHOT
kubectl apply -f deployment.yaml
kubectl port-forward svc/demo 8080:8080
```

And check your app:
```bash
curl localhost:8080/actuator/health
```


## References
* https://spring.io/guides/gs/spring-boot-kubernetes/
* https://kind.sigs.k8s.io/docs/user/local-registry/


