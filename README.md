# Kubernetes The Hard Way

본 튜토리얼은 Kubernetes 구성 과정을 명확하게 이해하기 위한 목적으로 Kubernetes를 별도의 부트스트랩(Kubeadm, kubespray) 없이, 설치를 직접하면서 추상화된 부분을 명확하게 이해하고자 한다.

## Copyright

<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="Creative Commons License" style="border-width:0" src="https://i.creativecommons.org/l/by-nc-sa/4.0/88x31.png" /></a><br />This work is licensed under a <a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">Creative Commons Attribution-NonCommercial-ShareAlike 4.0 International License</a>.


## Target Audience

본 튜토리얼은 쿠버네티스의 기본 사항과 핵심 구성 요소가 어떻게 구성되는지 이해한다. 더 나아가 EKS 등의 관리형 쿠버네티스 서비스를 구성하는 방법의 기본적 개념을 이해할 수 있다.

## Cluster Details

* 본 `Kubernetes The Hard Way` 는 원본과 달리 amd64 기반의 Ubuntu22.04를 기본으로 사용한다.
* 컴퓨팅 성능은 원본과 달리 모두 2코어 CPU와 2GB RAM으로 설정한다. (jumpbox 터미널 응답 속도가 늦으면, 더 하기 싫어지기 떄문이다.)
* CNI 구성은 원 구성과 다르게 Calico 로 하며, [Calico the hard way](https://docs.tigera.io/calico/latest/getting-started/kubernetes/hardway/) 튜토리얼을 토대로 구성한다. 


Component versions:

* [kubernetes](https://github.com/kubernetes/kubernetes) v1.28.x
* [containerd](https://github.com/containerd/containerd) v1.7.x
* [cni](https://github.com/containernetworking/cni) v1.3.x
* [etcd](https://github.com/etcd-io/etcd) v3.4.x

## Labs

* 이 튜토리얼은 4개의 amd64 아키텍처를 사용하는 Ubuntu 22.04, 2코어, 2GB RAM의 가상 머신으로 구성한다. 
* 가상 머신은 AWS, GCP, Azure, NHN Cloud 등 원하는 클라우드 제공업체나 VirtualBox 등을 사용해 생성한다.
  * (필자는 NHN Cloud의 직원이므로 NHN Cloud에서 가상 머신을 생성하여 튜토리얼을 진행한다.)

* [Prerequisites](docs/01-prerequisites.md)
* [Setting up the Jumpbox](docs/02-jumpbox.md)
* [Provisioning Compute Resources](docs/03-compute-resources.md)
* [Provisioning the CA and Generating TLS Certificates](docs/04-certificate-authority.md)
* [Generating Kubernetes Configuration Files for Authentication](docs/05-kubernetes-configuration-files.md)
* [Bootstrapping the etcd Cluster](docs/07-bootstrapping-etcd.md)
* [Bootstrapping the Kubernetes Control Plane](docs/08-bootstrapping-kubernetes-controllers.md)
* [Bootstrapping the Kubernetes Worker Nodes](docs/09-bootstrapping-kubernetes-workers.md)
* [Configuring kubectl for Remote Access](docs/10-configuring-kubectl.md)
* [Provisioning Pod Network Routes](docs/11-pod-network-routes.md)
* [Smoke Test](docs/12-smoke-test.md)
* [Cleaning Up](docs/13-cleanup.md)
