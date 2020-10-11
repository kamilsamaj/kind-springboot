# kind-springboot
Experimenting with Kind, SprintBoot, ServiceMesh, etc.

## Prerequisites
* curl
* git
* docker

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

## Create your KIND cluster
* Create local KIND k8s cluster
```bash
kind create cluster
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
