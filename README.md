# Instruction

## Abstract
- In the scope of this testing, we are going to deploy a minimal testing enviroment for Edge/IoT computing. There will be a simulated environment as a DataCenter where all the information of Edge Computing return, a simulated environment for Edge/IoT Data Server where IoT sensors running.

- At simulated DataCenter, there will be a GUI to display information sent from IoT sensors running on Edge. All the changes at the edge will be displayed on this GUI.

- In the next phase of testing, there will be the process of CI/CD on DataCenter or running AI/ML at the edge.

## Prerequisites
- There needs to have two Red Hat Openshift Cluster:
  * Cluster 1: Factory Datacenter (it can be used for CI/CD, Testing env in the next phase)
  * Cluster 2: Edge/IoT Data Server

## Cloning necessary repos in both of environment
- Clone the repo: https://github.com/vietstacker/manuela-dev.git
- Clone the repo: https://github.com/vietstacker/manuela-gitops.git

## On the Cluster 1 (Factory Datacenter)
- Change the cluster hostname. Grep the existing hostname from github (as an example) and replace it by your cluster name. Run the below command line and replace you cluster name in all the files except files located in "/manuela-tst/" directory
```bash
cd ~/manuela-gitops
grep -r "sing-e23f" .
```

- Subscribe to neccessary operators.
``` bash
cd ~/manuela-dev
oc apply -k namespaces_and_operator_subscriptions/openshift-pipelines
oc apply -k namespaces_and_operator_subscriptions/manuela-ci
oc apply -k namespaces_and_operator_subscriptions/argocd
oc apply -k namespaces_and_operator_subscriptions/manuela-temp-amq
```

- Wait for ArgoCD operator to be subscribed and available
```bash
oc get pods -n argocd

NAME                                             READY   STATUS              RESTARTS   AGE
argocd-operator-6d5d7bf9c5-kppd6                 1/1     Running             0          4d18h
```
- Create an ArgoCD instance and assign the cluster-admin role to its serviceaccount

```bash
oc apply -k infrastructure/argocd
oc adm policy add-cluster-role-to-user cluster-admin -n argocd -z argocd-application-controller
```

- Wait for the argocd secret to be created
```bash
oc get secret argocd-secret -n argocd

NAME            TYPE     DATA   AGE
argocd-secret   Opaque   2      30s
```

- Check the ArgoCD up and running

```bash
oc get pods -n argocd

NAME                                             READY   STATUS    RESTARTS   AGE
argocd-application-controller                    1/1     Running   0          1m
argocd-dex-server                                1/1     Running   0          1m
argocd-redis                                     1/1     Running   0          1m
argocd-repo-server                               1/1     Running   0          1m
argocd-server                                    1/1     Running   0          1m
```

- Deploy the Factory Data Center

```bash
oc create -n argocd -f ~/manuela-gitops/meta/argocd-ocp3.yaml
```
Note: In this scope, Red Hat OpenDataHub (ODH) and CI/CD is not used. You can see the .orig files in the ~/manuela-gitops/deployment/execenv-ocp3

- Remove manuela-temp-amq namespace

This namespace was created to kickstart the ArgoCD deployment of manuela-tst-all by making the AMQ Broker CRD known to the cluster. It can now be removed:
```bash
oc delete -k namespaces_and_operator_subscriptions/manuela-temp-amq
```

## On the Cluster 2 (Edge/IoT Data Server)

- Change the cluster hostname. Grep the existing hostname from github (as an example) and replace it by your cluster name. Run the below command line and replace you cluster name in all the files except files located in "/manuela-tst/" directory
```bash
cd ~/manuela-gitops
grep -r "sing-e23f" .
```

- Deploy ArgoCd
```bash
cd ~/manuela
oc apply -k namespaces_and_operator_subscriptions/argocd
oc apply -k infrastructure/argocd
```

- Deploy sensor services
```bash
oc apply -n argocd -f ~/manuela-gitops/meta/argocd-ocp4.yaml
```

- Remove manuela-temp-amq namespace
```bash
oc delete -k namespaces_and_operator_subscriptions/manuela-temp-amq
```
