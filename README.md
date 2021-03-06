
[![Build Status](https://travis-ci.org/sbezverk/nfproxy.svg?branch=master)](https://travis-ci.org/sbezverk/nfproxy)

<p align="left">
  <img src="https://github.com/sbezverk/nfproxy/blob/master/Logo_final.png?raw=true" width="40%" height="40%">
</p>

## kubernetes proxy functionality based on nftables

## Goal

The goal of nfproxy is to provide high performance and scalable kubernetes proxy supporting both ipv4 and ipv6. 
**nfproxy** is not a 1:1 copy of kube-proxy (iptables) in terms of features. **nfproxy** is not going to cover all corner
cases and special features addressed by kube-proxy if these features compromise the design principle of nfproxy which is

**"There is no rules per service or per endpoint"**. 

Meaning that the number of rules in one chain will not correlate to a number of services or endpoints.

This principle will limit applications of nfproxy, but on the other hand for the cases where nfproxy
can be used, it will offer superior performance and scalability when comparing with kube-proxy (iptables) implementation.

## Build

To build nfproxy binary execute:

```
make nfproxy

```
Resulting binary will be placed in *./bin* folder.

To build a container:

```
make container IMAGE_VERSION=X.X.X REGISTRY_NAME=docker.io/somename
```
This command will compile nfproxy binary and then will build a docker container tagged with
**REGISTRY_NAME/nfproxy:IMAGE_VERSION** and placed it in a local docker image store.

## Deployment

1. Find a way to save kube-proxy's daemonset yaml, once you tired of playing with nfproxy,
this yaml will allow you to restore the default kube-proxy functionality.

2. Delete kube-proxy daemonset and clean up iptables entries if kube-proxy ran in iptables mode

```
kubectl delete daemonset -n kube-system kube-proxy

sudo iptabes -F -t nat

sudo iptabes -F -t fiter
```

3. Modify nfproxy deployment yaml file to specify your cluster's CIDR and location of nfproxy image if not default
is used. 
**nfproxy** deployment file is located at ./deployment/nfproxy.yaml.

Change:
```
- "57.112.0.0/12"
```

For your cluster's cidr range.
```
- "X.Y.Z.0/L"
```
Where *L* is length in bits of your cluster's cidr.

4. Deploy nfproxy

```
kubectl create -f ./deployment/nfproxy.yaml
```

5. Check nfproxy pod's log

```
kubectl logs -n kube-system nfproxy-blah
```
If nfproxy started successfully, pod's log will contain messages about discovered services.

6. To delete nfproxy

```
kubectl delete -f ./deployment/nfproxy.yaml
```

## Status

Each nfproxy's PR is tested against kubernetes sig-network's e2e testing framework. 
The command line to run tests is the following:
```
 ./bazel-bin/test/e2e/e2e.test  -ginkgo.focus="\[sig-network\].*Service" -kubeconfig={location of kubeconfig file} -dns-domain={cluster's domain name}
```
The current status is the following:

```
[Fail] [sig-network] Services [It] should be able to switch session affinity for service with type clusterIP [LinuxOnly] [Flaky] 
test/e2e/network/service.go:193

[Fail] [sig-network] Services [It] should handle load balancer cleanup finalizer for service [Slow] 
test/e2e/framework/service/wait.go:78

[Fail] [sig-network] Services [It] should be able to switch session affinity for LoadBalancer service with ESIPP on [Slow] [DisabledForLargeClusters] [LinuxOnly] 
test/e2e/network/util.go:40

[Fail] [sig-network] Services [It] should be able to switch session affinity for NodePort service [LinuxOnly] [Flaky] 
test/e2e/network/service.go:193

[Fail] [sig-network] EndpointSlice [Feature:EndpointSlice] version v1 [It] should create Endpoints and EndpointSlices for Pods matching a Service 
test/e2e/network/endpointslice.go:358

[Fail] [sig-network] Services [It] should be able to switch session affinity for LoadBalancer service with ESIPP off [Slow] [DisabledForLargeClusters] [LinuxOnly] 
test/e2e/network/service.go:193

[Fail] [sig-network] Services [It] should have session affinity work for LoadBalancer service with ESIPP off [Slow] [DisabledForLargeClusters] [LinuxOnly] 
test/e2e/network/util.go:40

Ran 27 of 4846 Specs in 6224.795 seconds
FAIL! -- 20 Passed | 7 Failed | 0 Pending | 4819 Skipped
```

The effort is underway to make nfproxy to pass all e2e tests. Help is welcome and much appreciated.

Please file an issue for any discovered error or broken functionality.

**NOTE:** It is WIP, please expect rather high volume of changes.

**Contributors, reviewers, testers are welcome!!!**
