To get basic cluster info, switch into my cluster's context:

```
❯ kubectl config use-context redspot-test-a
Switched to context "redspot-test-a".
```

```
❯ kubectl cluster-info
Kubernetes master is running at https://redspot-test-a.kubernetes.us-east-1.prod.musta.ch
KubeDNS is running at https://redspot-test-a.kubernetes.us-east-1.prod.musta.ch/api/v1/namespaces/kube-system/services/kube-dns:dns/proxy
```

To get the nodes information:

```
❯ kubectl get nodes
NAME                             STATUS                     ROLES    AGE   VERSION
ip-172-21-155-62.ec2.internal    Ready                      <none>   36d   v1.13.12-airbnb0-release
ip-172-21-157-204.ec2.internal   Ready,SchedulingDisabled   <none>   8d    v1.13.12-airbnb0-release
ip-172-21-160-124.ec2.internal   Ready,SchedulingDisabled   <none>   8d    v1.13.12-airbnb0-release
ip-172-21-190-106.ec2.internal   Ready                      <none>   33d   v1.13.12-airbnb0-release
ip-172-21-229-156.ec2.internal   Ready,SchedulingDisabled   <none>   8d    v1.13.12-airbnb0-release
ip-172-21-240-74.ec2.internal    Ready                      <none>   36d   v1.13.12-airbnb0-release
```

Get a **very** comprehensive summary of the node you're interested in.
```
❯ kubectl describe node ip-172-21-155-62.ec2.internal
Name:               ip-172-21-155-62.ec2.internal
Roles:              <none>
Labels:             beta.kubernetes.io/arch=amd64
                    beta.kubernetes.io/instance-type=c5d.9xlarge
                    beta.kubernetes.io/os=linux
                    chef-role=kubernetes-minion-redspot-test-a
                    failure-domain.beta.kubernetes.io/region=us-east-1
                    failure-domain.beta.kubernetes.io/zone=us-east-1a
                    kubernetes.io/hostname=i-0a7beac1b868750e0
                    minion-type=normal
...
```

Enable auto-completion for kubectl related commands

```
❯ source <(kubectl completion bash)
```

Each pod in a worker node has its own IP address (resolved via kube-dns). But this address is internal to the cluster (i.e. I can't just ssh into it). How do I access it?

You want to expose it through a `Service` object. In this case, you want to create a `LoadBalancer` service which is external-facing and allows you to proxy through the load balancer to the pod.

```
❯ kubectl expose <resource> <app name> --type=LoadBalancer --name <name>
```

Note that kubernetes itself is also a `Service`:

```
❯ kubectl get services
NAME         TYPE        CLUSTER-IP   EXTERNAL-IP       PORT(S)   AGE
kubernetes   ClusterIP   100.68.0.1   <none>            443/TCP   49d
<lb name>    ...         ...          <IP to connect to>...       ...
```

Then you can connect to the IP shown in `EXTERNAL-IP`.

Why do you need services? because pods are ephemeral. They aren't reliable. A failing pod is replaced by a ReplicationController, which creates a new pod with a new internal IP. Kubernetes assumes you can't reliably connect to one pod, so it tells you the public IP of the load balancer, which is in charge of handling the constantly changing IP addresses of ephemeral pods. Services always have a static IP.

---

However, the above model has an issue with Jupyterhub:

1. It assumes each container in each pod are essentially the same, and stateless. This is not true for jupyterhub users, who like to install different things, use different docker images, etc.


---

Helm is a package manager which allows smooth deployment to install and upgrade kubernetes applications. Jupyterhub provides to us a bunch of **helm charts** which basically are templates for yaml configurations that we can fill in to have a concrete deployment.

To install Helm, run

```
❯ curl https://raw.githubusercontent.com/kubernetes/helm/master/scripts/get | bash
```

Our kubernetes cluster should already have Helm installed, so no need to run these commands (but are necessary on a cluster w/o helm):

```
❯ kubectl --namespace kube-system create serviceaccount tiller
❯ kubectl create clusterrolebinding tiller --clusterrole cluster-admin --serviceaccount=kube-system:tiller
❯ helm init --service-account tiller --history-max 100 --wait
```

To verify `helm` is installed on both and the versions are matching, do:

```
❯ helm version
Client: &version.Version{SemVer:"v2.16.6", GitCommit:"dd2e5695da88625b190e6b22e9542550ab503a47", GitTreeState:"clean"}
Server: &version.Version{SemVer:"v2.16.6", GitCommit:"dd2e5695da88625b190e6b22e9542550ab503a47", GitTreeState:"clean"}
```

---

To install Jupyterhub, we have to run:

```
❯ helm repo add jupyterhub https://jupyterhub.github.io/helm-chart/ && helm repo update
```

To run jupyterhub, create a repo with a `config.yaml`. If there is none, create it with the content:

```
proxy:
  secretToken: "something_here"
```

You should be able to run something like:

```
❯ helm install --name jupyterhub jupyterhub --values config.yaml
NAME:   jupyterhub
LAST DEPLOYED: Fri Apr 24 15:26:21 2020
NAMESPACE: default
STATUS: DEPLOYED

RESOURCES:
==> v1/ClusterRole
NAME                                     AGE
jupyterhub-user-scheduler-complementary  2s

==> v1/ClusterRoleBinding
NAME                                     AGE
jupyterhub-user-scheduler-base           2s
jupyterhub-user-scheduler-complementary  2s

==> v1/ConfigMap
NAME            DATA  AGE
hub-config      4     1s
user-scheduler  1     1s

==> v1/DaemonSet
NAME                     DESIRED  CURRENT  READY  UP-TO-DATE  AVAILABLE  NODE SELECTOR  AGE
continuous-image-puller  6        6        0      6           0          <none>         1s

==> v1/Deployment
NAME            READY  UP-TO-DATE  AVAILABLE  AGE
hub             0/1    1           0          1s
proxy           0/1    1           0          1s
user-scheduler  0/2    2           0          1s

==> v1/PersistentVolumeClaim
NAME        STATUS   VOLUME    CAPACITY  ACCESS MODES  STORAGECLASS  AGE
hub-db-dir  Pending  standard  1s

==> v1/Pod(related)

==> v1/Role
NAME  AGE
hub   2s

==> v1/RoleBinding
NAME  AGE
hub   2s

==> v1/Secret
NAME        TYPE    DATA  AGE
hub-secret  Opaque  2     1s

==> v1/Service
NAME          TYPE          CLUSTER-IP      EXTERNAL-IP  PORT(S)                     AGE
hub           ClusterIP     100.68.153.172  <none>       8081/TCP                    1s
proxy-api     ClusterIP     100.68.50.183   <none>       8001/TCP                    1s
proxy-public  LoadBalancer  100.68.18.222   <pending>    443:31580/TCP,80:30918/TCP  1s

==> v1/ServiceAccount
NAME            SECRETS  AGE
hub             1        1s
user-scheduler  1        1s

==> v1/StatefulSet
NAME              READY  AGE
user-placeholder  0/0    1s

==> v1beta1/PodDisruptionBudget
NAME              MIN AVAILABLE  MAX UNAVAILABLE  ALLOWED DISRUPTIONS  AGE
hub               1              N/A              0                    1s
proxy             1              N/A              0                    1s
user-placeholder  0              N/A              0                    1s
user-scheduler    1              N/A              0                    1s


NOTES:
Thank you for installing JupyterHub!

Your release is named jupyterhub and installed into the namespace default.

You can find if the hub and proxy is ready by doing:

 kubectl --namespace=default get pod

and watching for both those pods to be in status 'Running'.

You can find the public IP of the JupyterHub by doing:

 kubectl --namespace=default get svc proxy-public

It might take a few minutes for it to appear!

Note that this is still an alpha release! If you have questions, feel free to
  1. Read the guide at https://z2jh.jupyter.org
  2. Chat with us at https://gitter.im/jupyterhub/jupyterhub
  3. File issues at https://github.com/jupyterhub/zero-to-jupyterhub-k8s/issues
```
