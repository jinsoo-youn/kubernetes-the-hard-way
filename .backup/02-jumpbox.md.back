# Set Up The Jumpbox

In this lab you will set up one of the four machines to be a `jumpbox`. This machine will be used to run commands in this tutorial. While a dedicated machine is being used to ensure consistency, these commands can also be run from just about any machine including your personal workstation running macOS or Linux.

Think of the `jumpbox` as the administration machine that you will use as a home base when setting up your Kubernetes cluster from the ground up. One thing we need to do before we get started is install a few command line utilities and clone the Kubernetes The Hard Way git repository, which contains some additional configuration files that will be used to configure various Kubernetes components throughout this tutorial. 

Log in to the `jumpbox`:

```bash
ssh -i your/ssh/key ubuntu@jumpbox
```

Change to root user:

```bash
sudo su - 
```

All commands will be run as the `root` user. This is being done for the sake of convenience, and will help reduce the number of commands required to set everything up.

### Install Command Line Utilities

Now that you are logged into the `jumpbox` machine as the `root` user, you will install the command line utilities that will be used to preform various tasks throughout the tutorial. 

```bash
apt-get -y install wget curl vim openssl git
```

### Sync GitHub Repository

Now it's time to download a copy of this tutorial which contains the configuration files and templates that will be used build your Kubernetes cluster from the ground up. Clone the Kubernetes The Hard Way git repository using the `git` command:

```bash
git clone --depth 1 \
  https://github.com/jinsoo-youn/kubernetes-the-hard-way.git
```

Change into the `kubernetes-the-hard-way` directory:

```bash
cd kubernetes-the-hard-way
```

This will be the working directory for the rest of the tutorial. If you ever get lost run the `pwd` command to verify you are in the right directory when running commands on the `jumpbox`:

```bash
pwd
```

```text
/root/kubernetes-the-hard-way
```

### Download Binaries

In this section you will download the binaries for the various Kubernetes components. The binaries will be stored in the `downloads` directory on the `jumpbox`, which will reduce the amount of internet bandwidth required to complete this tutorial as we avoid downloading the binaries multiple times for each machine in our Kubernetes cluster.

From the `kubernetes-the-hard-way` directory create a `downloads` directory using the `mkdir` command:

```bash
mkdir downloads
```

The binaries that will be downloaded are listed in the `downloads.txt` file, which you can review using the `cat` command:

```bash
cat downloads.txt
```

Download the binaries listed in the `downloads.txt` file using the `wget` command:

```bash
wget -q --show-progress \
  --https-only \
  --timestamping \
  -P downloads \
  -i downloads.txt
```

인터넷 연결 속도에 따라 총 625 MB 의 바이너리 파일을 받는데 걸리는 시간이 다르며, 대략 24초 정도의 시간이 소요된다. 
다운로드가 끝나면, `ls` 명령어로 다운로드한 파일을 목록을 확인한다.

```bash
ls -loh downloads
```

```text
total 625M
-rw-r--r-- 1 root  44M May 10  2023 cni-plugins-linux-amd64-v1.3.0.tgz
-rw-r--r-- 1 root  46M Oct 27  2023 containerd-1.7.8-linux-amd64.tar.gz
-rw-r--r-- 1 root  23M Aug 14  2023 crictl-v1.28.0-linux-amd64.tar.gz
-rw-r--r-- 1 root  16M Jul 11  2023 etcd-v3.4.27-linux-amd64.tar.gz
-rw-r--r-- 1 root 117M Oct 18  2023 kube-apiserver
-rw-r--r-- 1 root 113M Oct 18  2023 kube-controller-manager
-rw-r--r-- 1 root  53M Oct 18  2023 kube-proxy
-rw-r--r-- 1 root  54M Oct 18  2023 kube-scheduler
-rw-r--r-- 1 root  48M Oct 18  2023 kubectl
-rw-r--r-- 1 root 106M Oct 18  2023 kubelet
-rw-r--r-- 1 root  11M Aug 11  2023 runc.amd64
```

### Install kubectl

In this section you will install the `kubectl`, the official Kubernetes client command line tool, on the `jumpbox` machine. `kubectl will be used to interact with the Kubernetes control once your cluster is provisioned later in this tutorial.

Use the `chmod` command to make the `kubectl` binary executable and move it to the `/usr/local/bin/` directory:

```bash
{
  chmod +x downloads/kubectl
  cp downloads/kubectl /usr/local/bin/
}
```

At this point `kubectl` is installed and can be verified by running the `kubectl` command:

```bash
kubectl version --client
```

```text
Client Version: v1.28.3
Kustomize Version: v5.0.4-0.20230601165947-6ce0bf390ce3
```

At this point the `jumpbox` has been set up with all the command line tools and utilities necessary to complete the labs in this tutorial.

Next: [Provisioning Compute Resources](03-compute-resources.md)
