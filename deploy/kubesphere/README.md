## Install Guide

### Table of contents
- [Preparation](#Preparation)
- [Usage](#Usage)
  - [Install](#install)
  - [Patch](#patch)
  - [Uninstall](#uninstall)
- [Access to the cluster](#access-to-the-cluster)
- [The Reference Configuration](#the-reference-configuration) 

### Preparation

Install helm 3.0+ and add repo:

```
$ helm repo add elastic https://helm.elastic.co
$ helm repo update
```

Clone codes from github:

```
$ git clone https://github.com/zhu33756/elastic-on-k8s
$ cd deploy/kubesphere
$ tree .
── Chart.yaml
├── Makefile
├── README.md
├── charts
│   └── eck-operator-crds
│       ├── Chart.yaml
│       ├── README.md
│       └── templates
│           ├── NOTES.txt
│           ├── _helpers.tpl
│           └── all-crds.yaml
├── es-cluster-with-auth.yaml
├── kubesphere-config.yaml
├── profile-global.yaml
├── profile-istio.yaml
├── profile-restricted.yaml
├── profile-soft-multi-tenancy.yaml
├── templates
│   ├── NOTES.txt
│   ├── _helpers.tpl
│   ├── cluster-roles.yaml
│   ├── configmap.yaml
│   ├── managed-namespaces.yaml
│   ├── managed-ns-network-policy.yaml
│   ├── operator-namespace.yaml
│   ├── operator-network-policy.yaml
│   ├── role-bindings.yaml
│   ├── service-account.yaml
│   ├── statefulset.yaml
│   ├── validate-chart.yaml
│   └── webhook.yaml
├── values-for-kubesphere.yaml
└── values.yaml
```

### Usage

We integrated the useful `make command line`, the description is as follows:

```
# Install
eg.  make install -e NAMESPACE=kubesphere-logging-system -e NAME_PREFFIX=kubesphere-eck

# Patch
eg. make patch

# Uninstall
eg. make uninstall

# Access to the cluster
eg. make access
```

#### Install

Before running `make install`, you can modify `values-for-kubesphere.yaml` && `es-cluster-with-auth.yaml` as what you want.

Note that `make install` cmdline would install eck-operator and es instance.

```
make install [-e NAMESPACE=xxx]
```

#### Patch

Note that `make patch` will integrate the es cluster on the kubesphere platform. Logs, audits, events would all save in the es cluster.

Once integrated, we can watch logs, audits, events by the toolbox on the platform.

```
make patch
```

#### Uninstall

This cmdline will unistall eck-operator, es cluster, and all the things related.

```
make uninstall 
```

### Access to the cluster

This cmdline will let us access to the cluster, it prints the steps.

```
$ make access

Step 1, run the following cmdline in anthor terminal:
        kubectl -n <namespace> port-forward service/xxx-eck-es-http 9200
Step 2, run the following cmdline in the current terminal:
        curl -u xxxx:xxxxx localhost:9200/_cluster/health?pretty
```

### The reference configuration

you can find the annotations in the files `es-cluster-with-auth.yaml` and `values-for-kubesphere.yaml`