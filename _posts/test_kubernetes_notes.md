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

---

We have these things called PVC (persistent volume claims):

```
❯ kubectl get pvc --namespace jhub
NAME         STATUS   VOLUME                                     CAPACITY   ACCESS MODES   STORAGECLASS   AGE
claim-asdf   Bound    pvc-6aca33fd-88e2-11ea-bd82-0af3cf9af059   10Gi       RWO            standard       22h
hub-db-dir   Bound    pvc-4dc23dcf-88d5-11ea-95f9-0e7c61f269ab   1Gi        RWO            standard       24h
```

---

To get the CA certs from a kube cluster:

```
❯ kubectl get secrets
NAME                  TYPE                                  DATA   AGE
awsecr-cred           kubernetes.io/dockerconfigjson        1      40d
default-token-qpk95   kubernetes.io/service-account-token   3      55d
```

To get the actual credentials:

```
❯ kubectl get secrets/default-token-qpk95 -oyaml
apiVersion: v1
data:
  ca.crt: LS0tLS1CRUdJTiBDRVJUSUZJQ0FURS0tLS0tCk1JSUYyekNDQThPZ0F3SUJBZ0lDRUFBd0RRWUpLb1pJaHZjTkFRRUxCUUF3Z1lNeEN6QUpCZ05WQkFZVEFsVlQKTVJNd0VRWURWUVFJREFwRFlXeHBabTl5Ym1saE1SWXdGQVlEVlFRSERBMVRZVzRnUm5KaGJtTnBjMk52TVJVdwpFd1lEVlFRS0RBeEJhWEppYm1Jc0lFbHVZeTR4RVRBUEJnTlZCQXNNQ0ZObFkzVnlhWFI1TVIwd0d3WURWUVFECkRCUkJhWEppYm1KSmJuUmxjbTVoYkZKdmIzUkRRVEFnRncweE5URXhNRGN3TURNME16TmFHQTh5TURnMU1UQXkKTURBd016UXpNMW93Y3pFTE1Ba0dBMVVFQmhNQ1ZWTXhFekFSQmdOVkJBZ01Da05oYkdsbWIzSnVhV0V4RlRBVApCZ05WQkFvTURFRnBjbUp1WWl3Z1NXNWpMakVSTUE4R0ExVUVDd3dJVTJWamRYSnBkSGt4SlRBakJnTlZCQU1NCkhFRnBjbUp1WWtsdWRHVnlibUZzU1c1MFpYSnRaV1JwWVhSbFEwRXdnZ0lpTUEwR0NTcUdTSWIzRFFFQkFRVUEKQTRJQ0R3QXdnZ0lLQW9JQ0FRRE1ZWXp0YkZvc0xyRkN3SU5aZjJOTUxWWVJPWExvM3lVNEo2UG1zdlJFQ1BTcQovMWkxakVuYklEVHpkaWhlZ2d2dWYva29JSDJoVG8wRE1vdno3TDBUMUZXUWtlNGV3QkZCbUxuYnhJblNpSUcvCjhqVDU2NXNmMGdKS0VmWU1pb1lud2orSTNKNklTUUY3TWExVEptVytkaVFRY0ZzaGlIdnQvcmp0UGVUS3gwSTYKSkk0SE42MVhJc2tEMlhBRHpZakVPMlE1L3ZReWFKOE9EaTgyQjlxdGNBT2xNUmdFa0UzdjNXcjArYTk5SzAregpKbFg0djRFem9POU5sN2NOTnplc3RwTVVuWlhDeWJUSzhPb0UxaDZPU05uUnJ3Sm1hOGxxMzAzektqWUZrZ3A4CmdXWjkyQ09yOTdqanYwSklBYk5lN3lvN21QeGYrMlhkUlBUcExURkFTME9aU2t1Mm9ZR2p1ODlieTJFVzA4algKTmlUckZrNTVFNzJaYzRnYmMrTUYvWWRVUHJBQVlZVW1paCtPdEFOOGVSb0sweFlJbG9zNmxoWkRaVU9OTGVQcApnOUVYVjRZSGRGekNZSm1ISmEwVEhxV1FzTXdkcXBtSUdKSTRkYTJNYWZNQjZxckFsdXUySVM2emVkcGdoZGJXCktRdE04QlUwbmViYXVVd25uQXpkNFEyaDRTTHlINGlvYlI0Um1QRHkwNm1uei9yQmlYbjFCLzN5VXpOTjhEbTgKZ0NsYXB1SnNPOUF5NTZ1cHFhb3dQbGRJYkFqUUloeXZpTkZXc25KSDA0RDFjS0NKQ0FVS3RPTjhqUHdWQlphTAptVytkZkREdm1kVFRtSHdhcFhNaDZBQjl1Q0FJWFlxK1p4RzBRQnBEREhOTEtZN3MvNzJ3UjBKOUdKRHh0d0lECkFRQUJvMll3WkRBZEJnTlZIUTRFRmdRVWRaMHJMRDd6dDBobXhLcG8zbmFOWW1KT3dTRXdId1lEVlIwakJCZ3cKRm9BVUF3L296dzZFUFZ0TUplcTRiZ1ovMEdESENVY3dFZ1lEVlIwVEFRSC9CQWd3QmdFQi93SUJBREFPQmdOVgpIUThCQWY4RUJBTUNBWVl3RFFZSktvWklodmNOQVFFTEJRQURnZ0lCQUNvbjBFV3pCRi9HWFAycFJkZUphVmRHCmFWRlVXRzVFT01lT1hFL00rc1pnVis4YjN5cmpKM0FNVTJaZStSdGptKzZ4N0dtMW5ScUFHNnBiVG5MSzJSREUKK2JKengzUnp3dWtvdFhzSHZ6Y0xvTFhmNTZteVRrRFBmcVZPYzVLTzJJVHRDYnVlNGtXS2UrdDRkbFUvQzJMbwpURy9oZXg0blRuMlVVY1poSlJDRm9rV0RaeFNDTXJXVGlyWVB3NTBxd2ZsKzMvNWhZb2I4YVh2ZkJPeTRtVnNICkFCN24waG5KUjIrZDA1ZXk3S0N4bklrRmk4R0lKcWtVVkdORG5XMGJXN3B4cEZEUjNyZnMzQnllaHp6ajNseU4KZElXSXB2TXFaTTVIQ0tsOVc1L0RBazlNRW1RQlNsa0tHRUpqVU1CMXN0Vk1PY3BZZEFWTHU1dlpVVnZnMFMyKwpkdFFia1ZSbFBqYXdZRHBVSWRNQ1dIK09CSm0wcDNRcVI3WXppZll1YnVEL2RRYjNSQU9jbnRiRkZNdkt3V3dwCjVOa3JyYzBBb0JnR3NGcnR3d1E5T0dXMkFaRVdVUlNjeCt6QkhVaTRPRVpFc0pjZjMrKzRUa2NpenMxNi9RN1EKaHBDSWxNSnFMUmlPeXZhYUYrdnRsT0hrY3NYVTRGbTNpY3AzRDhTQVVtdGQvUEV6S2tyUm9rWFdacExCUCtyVgpUWlJiYnBwQVROKy92czBCeEZIOGkzN2NrUlVaUHhienRUUTBwYUEvdDN1MHdSUHoxWHl0RjN3TnpHcnI5RzFnCkZiSzlNMDA2RnNLVENzTjBtd3A1eFlEcy9XWXFwQzNuaUY5VWNOVk1nWjJFZE9EaVhscFFqd01jWFVNdnhiMVgKZ0lUQnk1QldPK2FjemRSTmxhNUIKLS0tLS1FTkQgQ0VSVElGSUNBVEUtLS0tLQotLS0tLUJFR0lOIENFUlRJRklDQVRFLS0tLS0KTUlJRjhEQ0NBOWlnQXdJQkFnSUpBSzQwcStlK0ZvNVZNQTBHQ1NxR1NJYjNEUUVCQ3dVQU1JR0RNUXN3Q1FZRApWUVFHRXdKVlV6RVRNQkVHQTFVRUNBd0tRMkZzYVdadmNtNXBZVEVXTUJRR0ExVUVCd3dOVTJGdUlFWnlZVzVqCmFYTmpiekVWTUJNR0ExVUVDZ3dNUVdseVltNWlMQ0JKYm1NdU1SRXdEd1lEVlFRTERBaFRaV04xY21sMGVURWQKTUJzR0ExVUVBd3dVUVdseVltNWlTVzUwWlhKdVlXeFNiMjkwUTBFd0lCY05NVFV4TVRBM01EQXpOREl6V2hnUApNakV4TlRFd01UUXdNRE0wTWpOYU1JR0RNUXN3Q1FZRFZRUUdFd0pWVXpFVE1CRUdBMVVFQ0F3S1EyRnNhV1p2CmNtNXBZVEVXTUJRR0ExVUVCd3dOVTJGdUlFWnlZVzVqYVhOamJ6RVZNQk1HQTFVRUNnd01RV2x5WW01aUxDQkoKYm1NdU1SRXdEd1lEVlFRTERBaFRaV04xY21sMGVURWRNQnNHQTFVRUF3d1VRV2x5WW01aVNXNTBaWEp1WVd4UwpiMjkwUTBFd2dnSWlNQTBHQ1NxR1NJYjNEUUVCQVFVQUE0SUNEd0F3Z2dJS0FvSUNBUUN5dnk4ZXliWGJveitiCnk2dmk3UG9iSW4yM0hOZzM3L20yQXE5NzBwVzBKTllFU3g0YVRBVXNKNzlqdG92WjRZdGJrWWtHRUl6RjVlZEUKeE9ndzR5ZWZZUkovaFFrMlczcktqb0wxUXNJOUY4cHBHU1RkNU91NFpydWZmYVBqUDI4TGt5QU1CV29uaFBrTApTZlZrZFQxRlhkeWdWNUZsbjJxa3JVMytwaG1SVXNkclZ1cTdNM1J2b0NJVHFlQ05EekdOTmV1U1ZoU3FLaTVtCmFERktYckN0cmtNMnhJb1dUMCtWRDBwZ1pFTDhZV0kzSlhJZ1YvdWs3YWpZV1hVcVNib09Nbm9xbGI5ditmVzAKN2N0ZnpING52Rkk4MjYyZFJ1a1hXbk9HRXBPMWlrMkhLdURVaW1QWFM3ejA1UnlZbW5ReVVVRHZUWHF4K2s2NAo2VnpmbWZVMnZxakc4OE1Sc0tOdmJLd3k2TGJyUVpVQTh2cnlIZ0NlbmJhdmhxRmtRT0s1VzdPSk5udzN6TG5uCllpTGRIdG9tQWhjTEw0bXZRU0ZQd3dRNHdkOEYzWXF6ZG9wMVF4aDl6anJCeXA5TXErZ0RvWERTZ1BUdVVjYnMKWFJOdWZOQndZekhVdURmUUNmck5JOUNqMVJLQkd6bHRvTEJhUzdQZjF3TEJPRDRDaVJsdlRBRmR5L2JSSEZoOQpPTlhBeUhXSWswbDVhZXpLbGJHVVdYdnJPNEZ2eVFpTEJ0UkVCL1U1WVlsRXBraWJ3Z01XVUFKVzJFZDNZYkUrCnBGZmRJQUFXWE13NUMzMEFSZ0F4NSt2Wk01ajVNUDNuWTk5OG84azRWalNhZFlsODJBYWNEK3BFWDR1V3djUzcKYVc3NG5KUXN0REV1MGpUYkNtK2V3SXBLZ3BPYlVRSURBUUFCbzJNd1lUQWRCZ05WSFE0RUZnUVVBdy9venc2RQpQVnRNSmVxNGJnWi8wR0RIQ1Vjd0h3WURWUjBqQkJnd0ZvQVVBdy9venc2RVBWdE1KZXE0YmdaLzBHREhDVWN3CkR3WURWUjBUQVFIL0JBVXdBd0VCL3pBT0JnTlZIUThCQWY4RUJBTUNBWVl3RFFZSktvWklodmNOQVFFTEJRQUQKZ2dJQkFIU1FzVXl5TDNUK2dlU3lwNi8zRTg1RXlWVG1URFF3cjdGZW90czI4SW9MUGMvSVNqZHJjcXp5SXVZYwp5N2Era1kvUmh3ZjBSdnJ1MDZCWDg3cktNcTk1eEI0M253Z3ZHR0liZkd1ZzdoU1c3NmFnUEMvS0VKT1VhTVp4CkZIUkZMMi9nQ1VPcUp1bklQMnBmVnlzUTJhTW1ndUNDdnBlT1VnR29QWGRxVGdmaXdUbFlqKy9lRkdXenRQOS8KZkZNYk9JQkR6OWJsNk1ybVhIK2JDVzFNTFF0TWJvNFlNMXEzZW81Uis2V1lQSHFVellXRjN1QTRpTzlWTFBlawo5YXBkUFZqRXdnYjlMUW1rZ3hxdHJWazVacVRSSzlHVUxsdWRFdktEVkNnSWJkZnlFemkwbS9hazN1c1ZpSDhyCjc0U3dOSW5OeDJBMDY3d3VSNUhoWjdYYTEzVkViK0FtNkNORStVajZlbVdFaGhuRGJnSDU3T09yYWJkR1N5bVIKYUN0SW1NQnRWQ2F6b2ROZ096Zmk4OEREUmRmL2dnMmpzMUYxcnU4UGlNR2dYWWlSUnEvTHYyNFU1VDkyS3VhUgoyRHhadndXRjcrcnZldCsxaldkLytqMFEyKzlaVWdYRTllMmhWcEtoYjJCYzFWbmE0TFJlQmd0eUlpQzFvR09CCkd3bnJjdWNFTE5iM2h0MERiR0ZGMExQZlRFdmJydnEySGtDNHErdGoyUkU1aWdSR0FpVUxUaUt1SGJRSHJVVUgKd1RtQzhubU51b1RsQXVFMlJDK0Fhb2VXWTBUZWpDa0srS0U5NXdIS2hjSWJwVkFublVYQk1YSnkvYTZybThxMwo3eXd2T1QvVkxtQTNRNG1rTW5vTzE1K1FDRlhiVXE4OGVESEJYTUxReVN2UzIxbnAKLS0tLS1FTkQgQ0VSVElGSUNBVEUtLS0tLQo=
  namespace: ZGVmYXVsdA==
  token: ZXlKaGJHY2lPaUpTVXpJMU5pSXNJbXRwWkNJNklpSjkuZXlKcGMzTWlPaUpyZFdKbGNtNWxkR1Z6TDNObGNuWnBZMlZoWTJOdmRXNTBJaXdpYTNWaVpYSnVaWFJsY3k1cGJ5OXpaWEoyYVdObFlXTmpiM1Z1ZEM5dVlXMWxjM0JoWTJVaU9pSmtaV1poZFd4MElpd2lhM1ZpWlhKdVpYUmxjeTVwYnk5elpYSjJhV05sWVdOamIzVnVkQzl6WldOeVpYUXVibUZ0WlNJNkltUmxabUYxYkhRdGRHOXJaVzR0Y1hCck9UVWlMQ0pyZFdKbGNtNWxkR1Z6TG1sdkwzTmxjblpwWTJWaFkyTnZkVzUwTDNObGNuWnBZMlV0WVdOamIzVnVkQzV1WVcxbElqb2laR1ZtWVhWc2RDSXNJbXQxWW1WeWJtVjBaWE11YVc4dmMyVnlkbWxqWldGalkyOTFiblF2YzJWeWRtbGpaUzFoWTJOdmRXNTBMblZwWkNJNklqTmlZamRtT1Rka0xUVm1NalV0TVRGbFlTMDRORGc1TFRCaFlqazNOV1kyT1RFMlppSXNJbk4xWWlJNkluTjVjM1JsYlRwelpYSjJhV05sWVdOamIzVnVkRHBrWldaaGRXeDBPbVJsWm1GMWJIUWlmUS54R0U1V3ljTnI0VzhlMC1icFJpSXpkNW11R0toRGdQR0NrRTVRVGc1ZFlULW5DTWZaMjg1enJJWGh2YlN5MjhlOERlRkV0V1MwMkhhTUFfSFAxT0p2ai14VG0yVHd1ZGNFNm1wR2lLcVVVUmphS2VxVHk3Z0RfSXRuSVVrOGxreC1Fb2J0c2ZFUnJEbi1wTEdKREFzYmpuellyYUN1eVFkRFczQ0RPOHA5aGhRdjh2WmNIc20zMkNyeE5WOWNWWnVNQlZ0NU51QlpiX0F6MnpjVDBTTWhxYm01QnNicHNFaEdkekt5NWJ1Z19pbVBjdTQ0WGtTbzNVeFpKLTNzb29IR2ZkbzZuWkZfd2lZMmhLQlBhaC05SG9sWThvczBmYXFtcTBfUjBxaUZXMUdEZGplQXlfXzVxZDZZeEdvdTVlOVQ1YjVCMTNmWGNTd05xN0hfQ3VEYkE=
kind: Secret
metadata:
  annotations:
    kubernetes.io/service-account.name: default
    kubernetes.io/service-account.uid: 3bb7f97d-5f25-11ea-8489-0ab975f6916f
  creationTimestamp: "2020-03-05T21:06:50Z"
  name: default-token-qpk95
  namespace: default
  resourceVersion: "218"
  selfLink: /api/v1/namespaces/default/secrets/default-token-qpk95
  uid: 3bb8b899-5f25-11ea-8489-0ab975f6916f
type: kubernetes.io/service-account-token
```





```
apiVersion: v1
data:
  ca.crt: LS0tLS1CRUdJTiBDRVJUSUZJQ0FURS0tLS0tCk1JSUYyekNDQThPZ0F3SUJBZ0lDRUFBd0RRWUpLb1pJaHZjTkFRRUxCUUF3Z1lNeEN6QUpCZ05WQkFZVEFsVlQKTVJNd0VRWURWUVFJREFwRFlXeHBabTl5Ym1saE1SWXdGQVlEVlFRSERBMVRZVzRnUm5KaGJtTnBjMk52TVJVdwpFd1lEVlFRS0RBeEJhWEppYm1Jc0lFbHVZeTR4RVRBUEJnTlZCQXNNQ0ZObFkzVnlhWFI1TVIwd0d3WURWUVFECkRCUkJhWEppYm1KSmJuUmxjbTVoYkZKdmIzUkRRVEFnRncweE5URXhNRGN3TURNME16TmFHQTh5TURnMU1UQXkKTURBd016UXpNMW93Y3pFTE1Ba0dBMVVFQmhNQ1ZWTXhFekFSQmdOVkJBZ01Da05oYkdsbWIzSnVhV0V4RlRBVApCZ05WQkFvTURFRnBjbUp1WWl3Z1NXNWpMakVSTUE4R0ExVUVDd3dJVTJWamRYSnBkSGt4SlRBakJnTlZCQU1NCkhFRnBjbUp1WWtsdWRHVnlibUZzU1c1MFpYSnRaV1JwWVhSbFEwRXdnZ0lpTUEwR0NTcUdTSWIzRFFFQkFRVUEKQTRJQ0R3QXdnZ0lLQW9JQ0FRRE1ZWXp0YkZvc0xyRkN3SU5aZjJOTUxWWVJPWExvM3lVNEo2UG1zdlJFQ1BTcQovMWkxakVuYklEVHpkaWhlZ2d2dWYva29JSDJoVG8wRE1vdno3TDBUMUZXUWtlNGV3QkZCbUxuYnhJblNpSUcvCjhqVDU2NXNmMGdKS0VmWU1pb1lud2orSTNKNklTUUY3TWExVEptVytkaVFRY0ZzaGlIdnQvcmp0UGVUS3gwSTYKSkk0SE42MVhJc2tEMlhBRHpZakVPMlE1L3ZReWFKOE9EaTgyQjlxdGNBT2xNUmdFa0UzdjNXcjArYTk5SzAregpKbFg0djRFem9POU5sN2NOTnplc3RwTVVuWlhDeWJUSzhPb0UxaDZPU05uUnJ3Sm1hOGxxMzAzektqWUZrZ3A4CmdXWjkyQ09yOTdqanYwSklBYk5lN3lvN21QeGYrMlhkUlBUcExURkFTME9aU2t1Mm9ZR2p1ODlieTJFVzA4algKTmlUckZrNTVFNzJaYzRnYmMrTUYvWWRVUHJBQVlZVW1paCtPdEFOOGVSb0sweFlJbG9zNmxoWkRaVU9OTGVQcApnOUVYVjRZSGRGekNZSm1ISmEwVEhxV1FzTXdkcXBtSUdKSTRkYTJNYWZNQjZxckFsdXUySVM2emVkcGdoZGJXCktRdE04QlUwbmViYXVVd25uQXpkNFEyaDRTTHlINGlvYlI0Um1QRHkwNm1uei9yQmlYbjFCLzN5VXpOTjhEbTgKZ0NsYXB1SnNPOUF5NTZ1cHFhb3dQbGRJYkFqUUloeXZpTkZXc25KSDA0RDFjS0NKQ0FVS3RPTjhqUHdWQlphTAptVytkZkREdm1kVFRtSHdhcFhNaDZBQjl1Q0FJWFlxK1p4RzBRQnBEREhOTEtZN3MvNzJ3UjBKOUdKRHh0d0lECkFRQUJvMll3WkRBZEJnTlZIUTRFRmdRVWRaMHJMRDd6dDBobXhLcG8zbmFOWW1KT3dTRXdId1lEVlIwakJCZ3cKRm9BVUF3L296dzZFUFZ0TUplcTRiZ1ovMEdESENVY3dFZ1lEVlIwVEFRSC9CQWd3QmdFQi93SUJBREFPQmdOVgpIUThCQWY4RUJBTUNBWVl3RFFZSktvWklodmNOQVFFTEJRQURnZ0lCQUNvbjBFV3pCRi9HWFAycFJkZUphVmRHCmFWRlVXRzVFT01lT1hFL00rc1pnVis4YjN5cmpKM0FNVTJaZStSdGptKzZ4N0dtMW5ScUFHNnBiVG5MSzJSREUKK2JKengzUnp3dWtvdFhzSHZ6Y0xvTFhmNTZteVRrRFBmcVZPYzVLTzJJVHRDYnVlNGtXS2UrdDRkbFUvQzJMbwpURy9oZXg0blRuMlVVY1poSlJDRm9rV0RaeFNDTXJXVGlyWVB3NTBxd2ZsKzMvNWhZb2I4YVh2ZkJPeTRtVnNICkFCN24waG5KUjIrZDA1ZXk3S0N4bklrRmk4R0lKcWtVVkdORG5XMGJXN3B4cEZEUjNyZnMzQnllaHp6ajNseU4KZElXSXB2TXFaTTVIQ0tsOVc1L0RBazlNRW1RQlNsa0tHRUpqVU1CMXN0Vk1PY3BZZEFWTHU1dlpVVnZnMFMyKwpkdFFia1ZSbFBqYXdZRHBVSWRNQ1dIK09CSm0wcDNRcVI3WXppZll1YnVEL2RRYjNSQU9jbnRiRkZNdkt3V3dwCjVOa3JyYzBBb0JnR3NGcnR3d1E5T0dXMkFaRVdVUlNjeCt6QkhVaTRPRVpFc0pjZjMrKzRUa2NpenMxNi9RN1EKaHBDSWxNSnFMUmlPeXZhYUYrdnRsT0hrY3NYVTRGbTNpY3AzRDhTQVVtdGQvUEV6S2tyUm9rWFdacExCUCtyVgpUWlJiYnBwQVROKy92czBCeEZIOGkzN2NrUlVaUHhienRUUTBwYUEvdDN1MHdSUHoxWHl0RjN3TnpHcnI5RzFnCkZiSzlNMDA2RnNLVENzTjBtd3A1eFlEcy9XWXFwQzNuaUY5VWNOVk1nWjJFZE9EaVhscFFqd01jWFVNdnhiMVgKZ0lUQnk1QldPK2FjemRSTmxhNUIKLS0tLS1FTkQgQ0VSVElGSUNBVEUtLS0tLQotLS0tLUJFR0lOIENFUlRJRklDQVRFLS0tLS0KTUlJRjhEQ0NBOWlnQXdJQkFnSUpBSzQwcStlK0ZvNVZNQTBHQ1NxR1NJYjNEUUVCQ3dVQU1JR0RNUXN3Q1FZRApWUVFHRXdKVlV6RVRNQkVHQTFVRUNBd0tRMkZzYVdadmNtNXBZVEVXTUJRR0ExVUVCd3dOVTJGdUlFWnlZVzVqCmFYTmpiekVWTUJNR0ExVUVDZ3dNUVdseVltNWlMQ0JKYm1NdU1SRXdEd1lEVlFRTERBaFRaV04xY21sMGVURWQKTUJzR0ExVUVBd3dVUVdseVltNWlTVzUwWlhKdVlXeFNiMjkwUTBFd0lCY05NVFV4TVRBM01EQXpOREl6V2hnUApNakV4TlRFd01UUXdNRE0wTWpOYU1JR0RNUXN3Q1FZRFZRUUdFd0pWVXpFVE1CRUdBMVVFQ0F3S1EyRnNhV1p2CmNtNXBZVEVXTUJRR0ExVUVCd3dOVTJGdUlFWnlZVzVqYVhOamJ6RVZNQk1HQTFVRUNnd01RV2x5WW01aUxDQkoKYm1NdU1SRXdEd1lEVlFRTERBaFRaV04xY21sMGVURWRNQnNHQTFVRUF3d1VRV2x5WW01aVNXNTBaWEp1WVd4UwpiMjkwUTBFd2dnSWlNQTBHQ1NxR1NJYjNEUUVCQVFVQUE0SUNEd0F3Z2dJS0FvSUNBUUN5dnk4ZXliWGJveitiCnk2dmk3UG9iSW4yM0hOZzM3L20yQXE5NzBwVzBKTllFU3g0YVRBVXNKNzlqdG92WjRZdGJrWWtHRUl6RjVlZEUKeE9ndzR5ZWZZUkovaFFrMlczcktqb0wxUXNJOUY4cHBHU1RkNU91NFpydWZmYVBqUDI4TGt5QU1CV29uaFBrTApTZlZrZFQxRlhkeWdWNUZsbjJxa3JVMytwaG1SVXNkclZ1cTdNM1J2b0NJVHFlQ05EekdOTmV1U1ZoU3FLaTVtCmFERktYckN0cmtNMnhJb1dUMCtWRDBwZ1pFTDhZV0kzSlhJZ1YvdWs3YWpZV1hVcVNib09Nbm9xbGI5ditmVzAKN2N0ZnpING52Rkk4MjYyZFJ1a1hXbk9HRXBPMWlrMkhLdURVaW1QWFM3ejA1UnlZbW5ReVVVRHZUWHF4K2s2NAo2VnpmbWZVMnZxakc4OE1Sc0tOdmJLd3k2TGJyUVpVQTh2cnlIZ0NlbmJhdmhxRmtRT0s1VzdPSk5udzN6TG5uCllpTGRIdG9tQWhjTEw0bXZRU0ZQd3dRNHdkOEYzWXF6ZG9wMVF4aDl6anJCeXA5TXErZ0RvWERTZ1BUdVVjYnMKWFJOdWZOQndZekhVdURmUUNmck5JOUNqMVJLQkd6bHRvTEJhUzdQZjF3TEJPRDRDaVJsdlRBRmR5L2JSSEZoOQpPTlhBeUhXSWswbDVhZXpLbGJHVVdYdnJPNEZ2eVFpTEJ0UkVCL1U1WVlsRXBraWJ3Z01XVUFKVzJFZDNZYkUrCnBGZmRJQUFXWE13NUMzMEFSZ0F4NSt2Wk01ajVNUDNuWTk5OG84azRWalNhZFlsODJBYWNEK3BFWDR1V3djUzcKYVc3NG5KUXN0REV1MGpUYkNtK2V3SXBLZ3BPYlVRSURBUUFCbzJNd1lUQWRCZ05WSFE0RUZnUVVBdy9venc2RQpQVnRNSmVxNGJnWi8wR0RIQ1Vjd0h3WURWUjBqQkJnd0ZvQVVBdy9venc2RVBWdE1KZXE0YmdaLzBHREhDVWN3CkR3WURWUjBUQVFIL0JBVXdBd0VCL3pBT0JnTlZIUThCQWY4RUJBTUNBWVl3RFFZSktvWklodmNOQVFFTEJRQUQKZ2dJQkFIU1FzVXl5TDNUK2dlU3lwNi8zRTg1RXlWVG1URFF3cjdGZW90czI4SW9MUGMvSVNqZHJjcXp5SXVZYwp5N2Era1kvUmh3ZjBSdnJ1MDZCWDg3cktNcTk1eEI0M253Z3ZHR0liZkd1ZzdoU1c3NmFnUEMvS0VKT1VhTVp4CkZIUkZMMi9nQ1VPcUp1bklQMnBmVnlzUTJhTW1ndUNDdnBlT1VnR29QWGRxVGdmaXdUbFlqKy9lRkdXenRQOS8KZkZNYk9JQkR6OWJsNk1ybVhIK2JDVzFNTFF0TWJvNFlNMXEzZW81Uis2V1lQSHFVellXRjN1QTRpTzlWTFBlawo5YXBkUFZqRXdnYjlMUW1rZ3hxdHJWazVacVRSSzlHVUxsdWRFdktEVkNnSWJkZnlFemkwbS9hazN1c1ZpSDhyCjc0U3dOSW5OeDJBMDY3d3VSNUhoWjdYYTEzVkViK0FtNkNORStVajZlbVdFaGhuRGJnSDU3T09yYWJkR1N5bVIKYUN0SW1NQnRWQ2F6b2ROZ096Zmk4OEREUmRmL2dnMmpzMUYxcnU4UGlNR2dYWWlSUnEvTHYyNFU1VDkyS3VhUgoyRHhadndXRjcrcnZldCsxaldkLytqMFEyKzlaVWdYRTllMmhWcEtoYjJCYzFWbmE0TFJlQmd0eUlpQzFvR09CCkd3bnJjdWNFTE5iM2h0MERiR0ZGMExQZlRFdmJydnEySGtDNHErdGoyUkU1aWdSR0FpVUxUaUt1SGJRSHJVVUgKd1RtQzhubU51b1RsQXVFMlJDK0Fhb2VXWTBUZWpDa0srS0U5NXdIS2hjSWJwVkFublVYQk1YSnkvYTZybThxMwo3eXd2T1QvVkxtQTNRNG1rTW5vTzE1K1FDRlhiVXE4OGVESEJYTUxReVN2UzIxbnAKLS0tLS1FTkQgQ0VSVElGSUNBVEUtLS0tLQo=
  namespace: ZGVmYXVsdA==
  token: ZXlKaGJHY2lPaUpTVXpJMU5pSXNJbXRwWkNJNklpSjkuZXlKcGMzTWlPaUpyZFdKbGNtNWxkR1Z6TDNObGNuWnBZMlZoWTJOdmRXNTBJaXdpYTNWaVpYSnVaWFJsY3k1cGJ5OXpaWEoyYVdObFlXTmpiM1Z1ZEM5dVlXMWxjM0JoWTJVaU9pSmtaV1poZFd4MElpd2lhM1ZpWlhKdVpYUmxjeTVwYnk5elpYSjJhV05sWVdOamIzVnVkQzl6WldOeVpYUXVibUZ0WlNJNkltUmxabUYxYkhRdGRHOXJaVzR0YTJoeWVtSWlMQ0pyZFdKbGNtNWxkR1Z6TG1sdkwzTmxjblpwWTJWaFkyTnZkVzUwTDNObGNuWnBZMlV0WVdOamIzVnVkQzV1WVcxbElqb2laR1ZtWVhWc2RDSXNJbXQxWW1WeWJtVjBaWE11YVc4dmMyVnlkbWxqWldGalkyOTFiblF2YzJWeWRtbGpaUzFoWTJOdmRXNTBMblZwWkNJNklqQmhNREV5WldGakxUVm1NalV0TVRGbFlTMWlZV0ZpTFRCbE56RmpNekE1Wm1GbFpDSXNJbk4xWWlJNkluTjVjM1JsYlRwelpYSjJhV05sWVdOamIzVnVkRHBrWldaaGRXeDBPbVJsWm1GMWJIUWlmUS5tcUxRM0daRi00TUNYZ29Idkl3cU94bGlOSkExVmE3d0lMREN4dG8tQ2h4SGFWQlJ5RHcxMXpPcy1hcEFqWlp1d1lWOTBxSjBxaDFHOFMyUkRUVzdua2VjR2kycHptbU56cmx0WnV1WTJsdVN3ZW1ydVd5cXV2UzIzZldpTmhJQ2JUM1Y5TGRUYkxKVEZzWkV3RFMzUVpDUlVXSnNkM0ZvYll2XzJUVldLYmtRanZSQTVaRFlhSzFDUDRuYUJES2dPQ2xvd1hxRnVNSExvaEp6eEpRWThSN1hRSHFmU2QwNFhIOG4wb2xneXVVejlKbEhhdnNsNzR0SE5fSEV2UHpwWW5JQWdEcVo1ejQ5WE0tb1cwdDdNZFJ5T2lUTmwzYV8wYURDQlFkdkJsMWRXdkk1aFNkYzJzV3ppODR3VUsxMjRjYVlSaGc0cG5wanpCa3VoSFFtMWc=
kind: Secret
metadata:
  annotations:
    kubernetes.io/service-account.name: default
    kubernetes.io/service-account.uid: 0a012eac-5f25-11ea-baab-0e71c309faed
  creationTimestamp: "2020-03-05T21:05:27Z"
  name: default-token-khrzb
  namespace: default
  resourceVersion: "218"
  selfLink: /api/v1/namespaces/default/secrets/default-token-khrzb
  uid: 0a028886-5f25-11ea-baab-0e71c309faed
type: kubernetes.io/service-account-token
```

Hub service account in test:

```
LS0tLS1CRUdJTiBDRVJUSUZJQ0FURS0tLS0tCk1JSUYyekNDQThPZ0F3SUJBZ0lDRUFBd0RRWUpLb1pJaHZjTkFRRUxCUUF3Z1lNeEN6QUpCZ05WQkFZVEFsVlQKTVJNd0VRWURWUVFJREFwRFlXeHBabTl5Ym1saE1SWXdGQVlEVlFRSERBMVRZVzRnUm5KaGJtTnBjMk52TVJVdwpFd1lEVlFRS0RBeEJhWEppYm1Jc0lFbHVZeTR4RVRBUEJnTlZCQXNNQ0ZObFkzVnlhWFI1TVIwd0d3WURWUVFECkRCUkJhWEppYm1KSmJuUmxjbTVoYkZKdmIzUkRRVEFnRncweE5URXhNRGN3TURNME16TmFHQTh5TURnMU1UQXkKTURBd016UXpNMW93Y3pFTE1Ba0dBMVVFQmhNQ1ZWTXhFekFSQmdOVkJBZ01Da05oYkdsbWIzSnVhV0V4RlRBVApCZ05WQkFvTURFRnBjbUp1WWl3Z1NXNWpMakVSTUE4R0ExVUVDd3dJVTJWamRYSnBkSGt4SlRBakJnTlZCQU1NCkhFRnBjbUp1WWtsdWRHVnlibUZzU1c1MFpYSnRaV1JwWVhSbFEwRXdnZ0lpTUEwR0NTcUdTSWIzRFFFQkFRVUEKQTRJQ0R3QXdnZ0lLQW9JQ0FRRE1ZWXp0YkZvc0xyRkN3SU5aZjJOTUxWWVJPWExvM3lVNEo2UG1zdlJFQ1BTcQovMWkxakVuYklEVHpkaWhlZ2d2dWYva29JSDJoVG8wRE1vdno3TDBUMUZXUWtlNGV3QkZCbUxuYnhJblNpSUcvCjhqVDU2NXNmMGdKS0VmWU1pb1lud2orSTNKNklTUUY3TWExVEptVytkaVFRY0ZzaGlIdnQvcmp0UGVUS3gwSTYKSkk0SE42MVhJc2tEMlhBRHpZakVPMlE1L3ZReWFKOE9EaTgyQjlxdGNBT2xNUmdFa0UzdjNXcjArYTk5SzAregpKbFg0djRFem9POU5sN2NOTnplc3RwTVVuWlhDeWJUSzhPb0UxaDZPU05uUnJ3Sm1hOGxxMzAzektqWUZrZ3A4CmdXWjkyQ09yOTdqanYwSklBYk5lN3lvN21QeGYrMlhkUlBUcExURkFTME9aU2t1Mm9ZR2p1ODlieTJFVzA4algKTmlUckZrNTVFNzJaYzRnYmMrTUYvWWRVUHJBQVlZVW1paCtPdEFOOGVSb0sweFlJbG9zNmxoWkRaVU9OTGVQcApnOUVYVjRZSGRGekNZSm1ISmEwVEhxV1FzTXdkcXBtSUdKSTRkYTJNYWZNQjZxckFsdXUySVM2emVkcGdoZGJXCktRdE04QlUwbmViYXVVd25uQXpkNFEyaDRTTHlINGlvYlI0Um1QRHkwNm1uei9yQmlYbjFCLzN5VXpOTjhEbTgKZ0NsYXB1SnNPOUF5NTZ1cHFhb3dQbGRJYkFqUUloeXZpTkZXc25KSDA0RDFjS0NKQ0FVS3RPTjhqUHdWQlphTAptVytkZkREdm1kVFRtSHdhcFhNaDZBQjl1Q0FJWFlxK1p4RzBRQnBEREhOTEtZN3MvNzJ3UjBKOUdKRHh0d0lECkFRQUJvMll3WkRBZEJnTlZIUTRFRmdRVWRaMHJMRDd6dDBobXhLcG8zbmFOWW1KT3dTRXdId1lEVlIwakJCZ3cKRm9BVUF3L296dzZFUFZ0TUplcTRiZ1ovMEdESENVY3dFZ1lEVlIwVEFRSC9CQWd3QmdFQi93SUJBREFPQmdOVgpIUThCQWY4RUJBTUNBWVl3RFFZSktvWklodmNOQVFFTEJRQURnZ0lCQUNvbjBFV3pCRi9HWFAycFJkZUphVmRHCmFWRlVXRzVFT01lT1hFL00rc1pnVis4YjN5cmpKM0FNVTJaZStSdGptKzZ4N0dtMW5ScUFHNnBiVG5MSzJSREUKK2JKengzUnp3dWtvdFhzSHZ6Y0xvTFhmNTZteVRrRFBmcVZPYzVLTzJJVHRDYnVlNGtXS2UrdDRkbFUvQzJMbwpURy9oZXg0blRuMlVVY1poSlJDRm9rV0RaeFNDTXJXVGlyWVB3NTBxd2ZsKzMvNWhZb2I4YVh2ZkJPeTRtVnNICkFCN24waG5KUjIrZDA1ZXk3S0N4bklrRmk4R0lKcWtVVkdORG5XMGJXN3B4cEZEUjNyZnMzQnllaHp6ajNseU4KZElXSXB2TXFaTTVIQ0tsOVc1L0RBazlNRW1RQlNsa0tHRUpqVU1CMXN0Vk1PY3BZZEFWTHU1dlpVVnZnMFMyKwpkdFFia1ZSbFBqYXdZRHBVSWRNQ1dIK09CSm0wcDNRcVI3WXppZll1YnVEL2RRYjNSQU9jbnRiRkZNdkt3V3dwCjVOa3JyYzBBb0JnR3NGcnR3d1E5T0dXMkFaRVdVUlNjeCt6QkhVaTRPRVpFc0pjZjMrKzRUa2NpenMxNi9RN1EKaHBDSWxNSnFMUmlPeXZhYUYrdnRsT0hrY3NYVTRGbTNpY3AzRDhTQVVtdGQvUEV6S2tyUm9rWFdacExCUCtyVgpUWlJiYnBwQVROKy92czBCeEZIOGkzN2NrUlVaUHhienRUUTBwYUEvdDN1MHdSUHoxWHl0RjN3TnpHcnI5RzFnCkZiSzlNMDA2RnNLVENzTjBtd3A1eFlEcy9XWXFwQzNuaUY5VWNOVk1nWjJFZE9EaVhscFFqd01jWFVNdnhiMVgKZ0lUQnk1QldPK2FjemRSTmxhNUIKLS0tLS1FTkQgQ0VSVElGSUNBVEUtLS0tLQotLS0tLUJFR0lOIENFUlRJRklDQVRFLS0tLS0KTUlJRjhEQ0NBOWlnQXdJQkFnSUpBSzQwcStlK0ZvNVZNQTBHQ1NxR1NJYjNEUUVCQ3dVQU1JR0RNUXN3Q1FZRApWUVFHRXdKVlV6RVRNQkVHQTFVRUNBd0tRMkZzYVdadmNtNXBZVEVXTUJRR0ExVUVCd3dOVTJGdUlFWnlZVzVqCmFYTmpiekVWTUJNR0ExVUVDZ3dNUVdseVltNWlMQ0JKYm1NdU1SRXdEd1lEVlFRTERBaFRaV04xY21sMGVURWQKTUJzR0ExVUVBd3dVUVdseVltNWlTVzUwWlhKdVlXeFNiMjkwUTBFd0lCY05NVFV4TVRBM01EQXpOREl6V2hnUApNakV4TlRFd01UUXdNRE0wTWpOYU1JR0RNUXN3Q1FZRFZRUUdFd0pWVXpFVE1CRUdBMVVFQ0F3S1EyRnNhV1p2CmNtNXBZVEVXTUJRR0ExVUVCd3dOVTJGdUlFWnlZVzVqYVhOamJ6RVZNQk1HQTFVRUNnd01RV2x5WW01aUxDQkoKYm1NdU1SRXdEd1lEVlFRTERBaFRaV04xY21sMGVURWRNQnNHQTFVRUF3d1VRV2x5WW01aVNXNTBaWEp1WVd4UwpiMjkwUTBFd2dnSWlNQTBHQ1NxR1NJYjNEUUVCQVFVQUE0SUNEd0F3Z2dJS0FvSUNBUUN5dnk4ZXliWGJveitiCnk2dmk3UG9iSW4yM0hOZzM3L20yQXE5NzBwVzBKTllFU3g0YVRBVXNKNzlqdG92WjRZdGJrWWtHRUl6RjVlZEUKeE9ndzR5ZWZZUkovaFFrMlczcktqb0wxUXNJOUY4cHBHU1RkNU91NFpydWZmYVBqUDI4TGt5QU1CV29uaFBrTApTZlZrZFQxRlhkeWdWNUZsbjJxa3JVMytwaG1SVXNkclZ1cTdNM1J2b0NJVHFlQ05EekdOTmV1U1ZoU3FLaTVtCmFERktYckN0cmtNMnhJb1dUMCtWRDBwZ1pFTDhZV0kzSlhJZ1YvdWs3YWpZV1hVcVNib09Nbm9xbGI5ditmVzAKN2N0ZnpING52Rkk4MjYyZFJ1a1hXbk9HRXBPMWlrMkhLdURVaW1QWFM3ejA1UnlZbW5ReVVVRHZUWHF4K2s2NAo2VnpmbWZVMnZxakc4OE1Sc0tOdmJLd3k2TGJyUVpVQTh2cnlIZ0NlbmJhdmhxRmtRT0s1VzdPSk5udzN6TG5uCllpTGRIdG9tQWhjTEw0bXZRU0ZQd3dRNHdkOEYzWXF6ZG9wMVF4aDl6anJCeXA5TXErZ0RvWERTZ1BUdVVjYnMKWFJOdWZOQndZekhVdURmUUNmck5JOUNqMVJLQkd6bHRvTEJhUzdQZjF3TEJPRDRDaVJsdlRBRmR5L2JSSEZoOQpPTlhBeUhXSWswbDVhZXpLbGJHVVdYdnJPNEZ2eVFpTEJ0UkVCL1U1WVlsRXBraWJ3Z01XVUFKVzJFZDNZYkUrCnBGZmRJQUFXWE13NUMzMEFSZ0F4NSt2Wk01ajVNUDNuWTk5OG84azRWalNhZFlsODJBYWNEK3BFWDR1V3djUzcKYVc3NG5KUXN0REV1MGpUYkNtK2V3SXBLZ3BPYlVRSURBUUFCbzJNd1lUQWRCZ05WSFE0RUZnUVVBdy9venc2RQpQVnRNSmVxNGJnWi8wR0RIQ1Vjd0h3WURWUjBqQkJnd0ZvQVVBdy9venc2RVBWdE1KZXE0YmdaLzBHREhDVWN3CkR3WURWUjBUQVFIL0JBVXdBd0VCL3pBT0JnTlZIUThCQWY4RUJBTUNBWVl3RFFZSktvWklodmNOQVFFTEJRQUQKZ2dJQkFIU1FzVXl5TDNUK2dlU3lwNi8zRTg1RXlWVG1URFF3cjdGZW90czI4SW9MUGMvSVNqZHJjcXp5SXVZYwp5N2Era1kvUmh3ZjBSdnJ1MDZCWDg3cktNcTk1eEI0M253Z3ZHR0liZkd1ZzdoU1c3NmFnUEMvS0VKT1VhTVp4CkZIUkZMMi9nQ1VPcUp1bklQMnBmVnlzUTJhTW1ndUNDdnBlT1VnR29QWGRxVGdmaXdUbFlqKy9lRkdXenRQOS8KZkZNYk9JQkR6OWJsNk1ybVhIK2JDVzFNTFF0TWJvNFlNMXEzZW81Uis2V1lQSHFVellXRjN1QTRpTzlWTFBlawo5YXBkUFZqRXdnYjlMUW1rZ3hxdHJWazVacVRSSzlHVUxsdWRFdktEVkNnSWJkZnlFemkwbS9hazN1c1ZpSDhyCjc0U3dOSW5OeDJBMDY3d3VSNUhoWjdYYTEzVkViK0FtNkNORStVajZlbVdFaGhuRGJnSDU3T09yYWJkR1N5bVIKYUN0SW1NQnRWQ2F6b2ROZ096Zmk4OEREUmRmL2dnMmpzMUYxcnU4UGlNR2dYWWlSUnEvTHYyNFU1VDkyS3VhUgoyRHhadndXRjcrcnZldCsxaldkLytqMFEyKzlaVWdYRTllMmhWcEtoYjJCYzFWbmE0TFJlQmd0eUlpQzFvR09CCkd3bnJjdWNFTE5iM2h0MERiR0ZGMExQZlRFdmJydnEySGtDNHErdGoyUkU1aWdSR0FpVUxUaUt1SGJRSHJVVUgKd1RtQzhubU51b1RsQXVFMlJDK0Fhb2VXWTBUZWpDa0srS0U5NXdIS2hjSWJwVkFublVYQk1YSnkvYTZybThxMwo3eXd2T1QvVkxtQTNRNG1rTW5vTzE1K1FDRlhiVXE4OGVESEJYTUxReVN2UzIxbnAKLS0tLS1FTkQgQ0VSVElGSUNBVEUtLS0tLQo=
  namespace: amh1Yg==
  token: ZXlKaGJHY2lPaUpTVXpJMU5pSXNJbXRwWkNJNklpSjkuZXlKcGMzTWlPaUpyZFdKbGNtNWxkR1Z6TDNObGNuWnBZMlZoWTJOdmRXNTBJaXdpYTNWaVpYSnVaWFJsY3k1cGJ5OXpaWEoyYVdObFlXTmpiM1Z1ZEM5dVlXMWxjM0JoWTJVaU9pSnFhSFZpSWl3aWEzVmlaWEp1WlhSbGN5NXBieTl6WlhKMmFXTmxZV05qYjNWdWRDOXpaV055WlhRdWJtRnRaU0k2SW1oMVlpMTBiMnRsYmkwMWNYUTRPQ0lzSW10MVltVnlibVYwWlhNdWFXOHZjMlZ5ZG1salpXRmpZMjkxYm5RdmMyVnlkbWxqWlMxaFkyTnZkVzUwTG01aGJXVWlPaUpvZFdJaUxDSnJkV0psY201bGRHVnpMbWx2TDNObGNuWnBZMlZoWTJOdmRXNTBMM05sY25acFkyVXRZV05qYjNWdWRDNTFhV1FpT2lJMFpHTXlaV0V6T0MwNE9HUTFMVEV4WldFdE9UVm1PUzB3WlRkak5qRm1Nalk1WVdJaUxDSnpkV0lpT2lKemVYTjBaVzA2YzJWeWRtbGpaV0ZqWTI5MWJuUTZhbWgxWWpwb2RXSWlmUS5oYWRfX2tjRndzcUR6Q1FZSG5MOF94SzNqWEZpdzZhekJZaVNSRndobUZ0eWM0YVh5NHNwVTJwOWdsclZHMHNlcHMyRWlLTFVnTGZVYTA2WEpSelAzYWxRNEFaWFUwZVJMdU80aHR0c25sMkx1bmNoSFhFZUI2RHRwWEpTbVE5WGlpYW5rREczZ2MxQTdIX3hxSllUR3d0S1MtMEZleVZsc0NOakxjODN0LV9OVU1XZU5ydTdQVFlUVnRKeWZ0aG1nSmtKMFpPMVF4TlRVVTUxNTdVMmQzcFJNUXI2dU1KTjhNQlRCTlVVZVZGejZ3NFNOQTRjNHFZeGdmdnhKREFuUklaU0c5M2NuRkM3WmdWdDFxYURQTkczWEk3aHpFX3dGWFQ1UWl2QkJqQmlZa3o4WlFDUWc5bUlib2dab1gwSm9Ub1lRRnlDSGd2NWpjQk9KTHNzanc=
```

<script src="https://utteranc.es/client.js" repo="OneRaynyDay/oneraynyday.github.io" issue-term="pathname" theme="github-light" crossorigin="anonymous" async> </script>
