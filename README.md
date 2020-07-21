# Kubernetes perf-tests

[![Build Status](https://travis-ci.org/kubernetes/perf-tests.svg?branch=master)](https://travis-ci.org/kubernetes/perf-tests)  [![Go Report Card](https://goreportcard.com/badge/github.com/kubernetes/perf-tests)](https://goreportcard.com/report/github.com/kubernetes/perf-tests)


This repo is dedicated for storing various Kubernetes-related performance test related tools. If you want to add your own load-test, benchmark, framework or other tool please contact with one of the Owners.

Because in general tools are independent and have their own ways of being configured or run, each subdirectory needs a separate README.md file with the description of its contents.

## Repository setup

To run all verify* scripts before pushing to remote branch (useful for catching problems quickly) execute:

```
cp _hook/pre-push .git/hooks/pre-push
```

# ClusterLoader

ClusterLoader can be found [here](https://github.com/kubernetes/perf-tests/tree/master/clusterloader2).

ClusterLoader is an open-source tool for testing the scale of Kubernetes components. 

## How to run ClusterLoader in a kubenetes cluster?

1. Download perf-tests repository https://github.com/kubernetes/perf-tests to a machine that has access to the kubemaster of the testing kubernets cluster, i.e., you can run kubectl to CRUD namespaces, Pods, etc. 

2. Run clusterloader: go run $GOPATH/src/k8s.io/perf-tests/clusterloader2/cmd/clusterloader.go. 

If you run clusterloader.go from a cluster machine, you should modify the command line parameters according to your configurations and needs, such as kubeconfig file, and test configuration file. 
      
```
#!/bin/sh
set -x

cd to perf-tests directory 
go run cmd/clusterloader.go \
        --nodes=2000 \
        --kubeconfig=~/.kube/kubeconfig \
        --provider=local \
        --testconfig=testing/density/config-test.yaml \
        --testoverride=testing/density/1500_nodes/override.yaml \
        --masterip=  \
        --mastername=kube-api \
        --master-internal-ip=kube-api \
        --kubeletport=10260  \
        --report-dir=/home/yuan_chen3/test/log
```
   
A script that launches a testing and an example kube configration file are included in script subdirectory. Below is an example kubeconfig file in config sub-directory.

```
apiVersion: v1
clusters:
- cluster:
    certificate-authority: /etc/certs/trusted-root.pem
    server: 
    name: default_cluster
contexts:
- context:
    cluster: default_cluster
    user: admin
  name: default_cluster
current-context: default_cluster
kind: Config
preferences: {}
users:
- name: admin
  user:
    client-certificate: /var/run/appcerts/kube-admin/application.crt
    client-key: /var/run/appcerts/kube-admin/application.key
```

## How to run scalability testing 

1. Download and copy the respistory to a directory (e.g., $CL) on kmaster001.usspk008.pie.apple.com.

2. ssh to a master node

3. The pre-created configuration templates and files are available in profile subdirectory. There are three pre-defined templates for launching deployments (template_deployment.yaml),  jobs (template_job.yaml) and services (template_service.yaml). To create a test case, you should choose a template file and override cusotimization prameters in a separate file in profile/override subdirctory.  A number of predefined test cases and overridden files can be found in  profile and prifile/override respectively. The directory name is self-explanatory. You can edit it or create your own overriden files. You can also modify or create new templates if necessary. Here are some configurable parameters. 

```
TOTAL_NODES: 1000
NAMESPACES: 1
DEPLOYMENT_TYPES: 1
SERVICE_TYPES: 1
JOB_TYPES: 1

#for each deployment/job type
REPLICAS_PER_DEPLOYMEN: 100
POD_CPU: 100m
POD_MEMORY: 128Mi 
OBJECT_FILE: "deployment.yaml"
IMAGE: "nginx:1.7.9"
PRIORITIES: "medium"
SCALING_DOWN_FACTOR: 1.5
SCALING_UP_FACTOR: 1.5
LOAD_GENERATION_MODE: "Qps"
ENABLE_SCALING_TEST: false
```

4. Run the following command to start a load test.

   $CL/example/testing/bin/run-test.sh [-k kubeconfig file] [-t testconfig file] [-o testoverride file] [-c clusterloader directory] [-l log directory] 

```
cmd="go run $clusterloader_dir/cmd/clusterloader.go \
        --kubeconfig=$kubeconfig \
        --testconfig=$testconfig \
        --testoverride=$testoverride \
        --masterip= ... \
        --mastername=kube-api \
        --master-internal-ip=kube-api \
        --kubeletport=10260  \
        --provider=local \
        --report-dir=$log_dir"
```

5. The output of a test is saved in the log directory. 


