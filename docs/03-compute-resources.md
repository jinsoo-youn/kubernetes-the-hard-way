# Provisioning Compute Resources

Kubernetes requires a set of machines to host the Kubernetes control plane and the worker nodes where containers are ultimately run. In this lab you will provision the machines required for setting up a Kubernetes cluster.

![alt text](image.png)

## Machine Database

This tutorial will leverage a text file, which will serve as a machine database, to store the various machine attributes that will be used when setting up the Kubernetes control plane and worker nodes. The following schema represents entries in the machine database, one entry per line:

```text
IPV4_ADDRESS FQDN HOSTNAME POD_SUBNET
```

Each of the columns corresponds to a machine IP address `IPV4_ADDRESS`, fully qualified domain name `FQDN`, host name `HOSTNAME`.

Here is an example machine database similar to the one used when creating this tutorial. Notice the IP addresses have been masked out. Your machines can be assigned any IP address as long as each machine is reachable from each other and the `jumpbox`.

```bash
cat << EOF > machines.txt
192.168.0.16 server.kubernetes.local server
192.168.0.32 node-0.kubernetes.local node-0
192.168.0.100 node-1.kubernetes.local node-1
EOF
```

```text
XXX.XXX.XXX.XXX server.kubernetes.local server  
XXX.XXX.XXX.XXX node-0.kubernetes.local node-0 10.200.0.0/24
XXX.XXX.XXX.XXX node-1.kubernetes.local node-1 10.200.1.0/24
```

Now it's your turn to create a `machines.txt` file with the details for the three machines you will be using to create your Kubernetes cluster. Use the example machine database from above and add the details for your machines. 
> 본문과 다르게 본 튜토리얼에서는 Calico를 통해 Container Network를 구성하기 때문에 별도의 각 node 별도 pod의 ip 범위를 machines.txt 파일에 명시할 필요가 없다.

## Configuring SSH Access

SSH will be used to configure the machines in the cluster. Verify that you have `root` SSH access to each machine listed in your machine database. You may need to enable root SSH access on each node by updating the sshd_config file and restarting the SSH server.
> root로 바로 접근이 허용되어 있으면 별도의 설정이 필요 없으나, 대부분 Cloud Provider 에서 root 로그인을 막고 있다. NHN Cloud도 동일하므로 root 접근이 가능하게 설정 변경이 필요하다.

### Enable root SSH Access

> server, node-0, node-1 의 각 노드에 접근하여 아래 제공된 명령어들을 실행하여 root 접근이 가능하게 설정한다. 
> 일반적으로 FIP를 연결한 jumpbox 에 접속 후 같은 VPC 내에 있는 VM으로 접속하여 설정을 한다.
> jumpbox 에서 VM 이름으로 편하게 접속하기 위해 우선적으로 /etc/hosts에 machines.txt 정보를 입력한다. 

```bash 
cat machines.txt >> /etc/hosts
cat /etc/hosts
```

```text
127.0.1.1 jumpbox.novalocal jumpbox
127.0.0.1 localhost

# The following lines are desirable for IPv6 capable hosts
::1 localhost ip6-localhost ip6-loopback
ff02::1 ip6-allnodes
ff02::2 ip6-allrouters

192.168.0.16 server.kubernetes.local server
192.168.0.32 node-0.kubernetes.local node-0
192.168.0.100 node-1.kubernetes.local node-1
```

추가로 모두 동일한 키페어를 사용했다는 가정하에 사용된 키페어를 `scp` 명령어로 `jumpbox` 에 옮긴다.

```bash
scp -i ~/.ssh/beta-jinsoo.pem ~/.ssh/beta-jinsoo.pem root@125.6.48.156:/root/.ssh
```

If `root` SSH access is enabled for each of your machines you can skip this section.

By default, a new `debian` install disables SSH access for the `root` user. This is done for security reasons as the `root` user is a well known user on Linux systems, and if a weak password is used on a machine connected to the internet, well, let's just say it's only a matter of time before your machine belongs to someone else. As mention earlier, we are going to enable `root` access over SSH in order to streamline the steps in this tutorial. Security is a tradeoff, and in this case, we are optimizing for convenience. On each machine login via SSH using your user account, then switch to the `root` user using the `su` command:

```bash
ssh -i ~/.ssh/beta-jinsoo.pem ubuntu@server
sudo su -
```

Edit the `/etc/ssh/sshd_config` SSH daemon configuration file and the `PermitRootLogin` option to `yes`:

```bash
sed -i \
  's/^PermitRootLogin.*/PermitRootLogin yes/' \
  /etc/ssh/sshd_config
```

Restart the `sshd` SSH server to pick up the updated configuration file:

```bash
systemctl restart sshd
```

### Generate and Distribute SSH Keys

In this section you will generate and distribute an SSH keypair to the `server`, `node-0`, and `node-1`, machines, which will be used to run commands on those machines throughout this tutorial. Run the following commands from the `jumpbox` machine.

Generate a new SSH key:

```bash
ssh-keygen
```

```text
Generating public/private rsa key pair.
Enter file in which to save the key (/root/.ssh/id_rsa): 
Enter passphrase (empty for no passphrase): 
Enter same passphrase again: 
Your identification has been saved in /root/.ssh/id_rsa
Your public key has been saved in /root/.ssh/id_rsa.pub
```

Copy the SSH public key to each machine:

수동 복사
1. 기존 키로 인스턴스에 접속:
```bash
ssh -i /path/to/your-key-pair.pem ubuntu@hostname

sudo su -
```

2. 로컬 머신에서 공개 키 확인:
```bash
cat ~/.ssh/id_rsa.pub
```

3. 인스턴스에서 authorized_keys 파일에 추가:
```bash
echo "ssh-rsa AAAAB3...your-public-key... root@hostname" >> ~/.ssh/authorized_keys
chmod 600 ~/.ssh/authorized_keys
```

4. 새 SSH 키로 접속 확인:  
```bash
ssh root@hostname
```

Once each key is added, verify SSH public key access is working:

```bash
while read IP FQDN HOST; do 
  ssh -n root@${IP} uname -o -m
done < machines.txt
```

```text
Please login as the user "ubuntu" rather than the user "root".
x86_64 GNU/Linux
Please login as the user "ubuntu" rather than the user "root".
x86_64 GNU/Linux
Please login as the user "ubuntu" rather than the user "root".
x86_64 GNU/Linux
```

## Hostnames

In this section you will assign hostnames to the `server`, `node-0`, and `node-1` machines. The hostname will be used when executing commands from the `jumpbox` to each machine. The hostname also play a major role within the cluster. Instead of Kubernetes clients using an IP address to issue commands to the Kubernetes API server, those client will use the `server` hostname instead. Hostnames are also used by each worker machine, `node-0` and `node-1` when registering with a given Kubernetes cluster.

To configure the hostname for each machine, run the following commands on the `jumpbox`.

Set the hostname on each machine listed in the `machines.txt` file:

```bash
while read IP FQDN HOST SUBNET; do 
    CMD="sed -i 's/^127.0.1.1.*/127.0.1.1\t${FQDN} ${HOST}/' /etc/hosts"
    ssh -n root@${IP} "$CMD"
    ssh -n root@${IP} hostnamectl hostname ${HOST}
done < machines.txt
```

Verify the hostname is set on each machine:

```bash
while read IP FQDN HOST SUBNET; do
  ssh -n root@${IP} hostname --fqdn
done < machines.txt
```

```text
server.kubernetes.local
node-0.kubernetes.local
node-1.kubernetes.local
```

## DNS

In this section you will generate a DNS `hosts` file which will be appended to `jumpbox` local `/etc/hosts` file and to the `/etc/hosts` file of all three machines used for this tutorial. This will allow each machine to be reachable using a hostname such as `server`, `node-0`, or `node-1`.

Create a new `hosts` file and add a header to identify the machines being added:

```bash
echo "" > hosts
echo "# Kubernetes The Hard Way" >> hosts
```

Generate a DNS entry for each machine in the `machines.txt` file and append it to the `hosts` file:

```bash
while read IP FQDN HOST; do 
    ENTRY="${IP} ${FQDN} ${HOST}"
    echo $ENTRY >> hosts
done < machines.txt
```

Review the DNS entries in the `hosts` file:

```bash
cat hosts
```

```text

# Kubernetes The Hard Way
XXX.XXX.XXX.XXX server.kubernetes.local server
XXX.XXX.XXX.XXX node-0.kubernetes.local node-0
XXX.XXX.XXX.XXX node-1.kubernetes.local node-1
```

## Adding DNS Entries To A Local Machine

In this section you will append the DNS entries from the `hosts` file to the local `/etc/hosts` file on your `jumpbox` machine.

Append the DNS entries from `hosts` to `/etc/hosts`:

```bash
cat hosts >> /etc/hosts
```

Verify that the `/etc/hosts` file has been updated:

```bash
cat /etc/hosts
```

```text
127.0.0.1       localhost
127.0.1.1       jumpbox

# The following lines are desirable for IPv6 capable hosts
::1     localhost ip6-localhost ip6-loopback
ff02::1 ip6-allnodes
ff02::2 ip6-allrouters



# Kubernetes The Hard Way
XXX.XXX.XXX.XXX server.kubernetes.local server
XXX.XXX.XXX.XXX node-0.kubernetes.local node-0
XXX.XXX.XXX.XXX node-1.kubernetes.local node-1
```

At this point you should be able to SSH to each machine listed in the `machines.txt` file using a hostname.

```bash
for host in server node-0 node-1
   do ssh root@${host} uname -o -m -n
done
```

```text
server aarch64 GNU/Linux
node-0 aarch64 GNU/Linux
node-1 aarch64 GNU/Linux
```

## Adding DNS Entries To The Remote Machines

In this section you will append the DNS entries from `hosts` to `/etc/hosts` on each machine listed in the `machines.txt` text file.

Copy the `hosts` file to each machine and append the contents to `/etc/hosts`:

```bash
while read IP FQDN HOST; do
  scp hosts root@${HOST}:~/
  ssh -n \
    root@${HOST} "cat hosts >> /etc/hosts"
done < machines.txt
```

At this point hostnames can be used when connecting to machines from your `jumpbox` machine, or any of the three machines in the Kubernetes cluster. Instead of using IP addresess you can now connect to machines using a hostname such as `server`, `node-0`, or `node-1`.

Next: [Provisioning a CA and Generating TLS Certificates](04-certificate-authority.md)
