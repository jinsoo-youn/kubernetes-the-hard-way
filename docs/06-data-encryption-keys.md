# Generating the Data Encryption Config and Key

kube-apiserver -> etcd 로 데이터 암호화는 본 튜토리얼에서 사실 의미가 없기 때문에 이 부분은 생략한다. 
`units/kube-apiserver.service` 파일에 `  --encryption-provider-config=/var/lib/kubernetes/encryption-config.yaml \` 항목을 제외하여 apiserver를 구성한다.

Kubernetes stores a variety of data including cluster state, application configurations, and secrets. Kubernetes supports the ability to [encrypt](https://kubernetes.io/docs/tasks/administer-cluster/encrypt-data) cluster data at rest.

In this lab you will generate an encryption key and an [encryption config](https://kubernetes.io/docs/tasks/administer-cluster/encrypt-data/#understanding-the-encryption-at-rest-configuration) suitable for encrypting Kubernetes Secrets.

## The Encryption Key

Generate an encryption key:

```bash
export ENCRYPTION_KEY=$(head -c 32 /dev/urandom | base64)
```

## The Encryption Config File

Create the `encryption-config.yaml` encryption config file:

```bash
envsubst < configs/encryption-config.yaml \
  > encryption-config.yaml
```

Copy the `encryption-config.yaml` encryption config file to each controller instance:

```bash
scp encryption-config.yaml root@server:~/
```

Next: [Bootstrapping the etcd Cluster](07-bootstrapping-etcd.md)
