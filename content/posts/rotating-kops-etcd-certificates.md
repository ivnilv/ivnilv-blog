---
title: "Rotating Kops Etcd Certificates"
date: 2020-12-08T18:27:01+02:00
draft: false
tags:
- Kubernetes
- Kops
- AWS
- etcd
---

Reading time: 5 min

---

#### *Check etcd-manager version*

Using kubectl:

```
$ k -n kube-system get pod etcd-manager-main-ip-NODE-IP-ADDRESS -o yaml | grep "image\:"
    image: kopeio/etcd-manager:3.0.20200429
```

According to the [releases documentation](https://github.com/kopeio/etcd-manager/releases) version 3.0.20200428 brings a fix that renews expiring certificates in the cluster.

However, the implementation of this as noted in github issue [#309](https://github.com/kopeio/etcd-manager/pull/309) 

> Not a perfect fix, if you don't restart things every now and then, they could still expire. But it's at least closer and means if you do restart things, it will fix itself.

#### *Rotating expired etcd client certificates*

After the etcd-manager running in the cluster is confirmed to be 3.0.20200428 or above, all that's needed to regenerate the certificates and restore the etcd cluster is restarting the Kubernetes master nodes. You could also restart the etcd main and etcd events pods instead, however we'll go with restarting the whole EC2 instances.

When the restarted instance comes back up it will start the `etcd-main` and `etcd-events` pods , which will trigger the startup checks implemented in the `etcd-manager` code to check the certificates expiration. 

If it finds that the certificates are expiring in 60 days or less, it will regenerate them. You should see the following output in the logs for the `etcd-main` and `etcd-events` pods:

```
etcd-manager
...
I1208 08:37:56.019184   12806 main.go:299] Setting data dir to /rootfs/mnt/master-vol-ID
I1208 08:37:56.019648   12806 certs.go:106] existing certificate not valid after 2022-12-08T08:37:05Z; will regenerate
I1208 08:37:56.019655   12806 certs.go:167] generating certificate for "etcd-manager-server-etcd-c"
I1208 08:37:56.023307   12806 certs.go:106] existing certificate not valid after 2022-12-08T08:37:05Z; will regenerate
I1208 08:37:56.023350   12806 certs.go:167] generating certificate for "etcd-manager-client-etcd-c"
...
I1208 08:37:56.032081   12806 pki.go:39] generating peer keypair for etcd: {CommonName:etcd-c Organization:[] AltNames:{DNSNames:[etcd-c.internal.clstrname.k8s.local] IPs:[127.0.0.1]} Usages:[2 1]}
I1208 08:37:56.032340   12806 certs.go:106] existing certificate not valid after 2022-12-08T08:37:05Z; will regenerate
I1208 08:37:56.032346   12806 certs.go:167] generating certificate for "etcd-c"
I1208 08:37:56.041521   12806 pki.go:79] building client-serving certificate: {CommonName:etcd-c Organization:[] AltNames:{DNSNames:[etcd-c.internal.clstrname.k8s.local etcd-c.internal.clstrname.k8s.local] IPs:[127.0.0.1 127.0.0.1]} Usages:[1 2]}
I1208 08:37:56.041786   12806 certs.go:106] existing certificate not valid after 2022-12-08T08:37:05Z; will regenerate
I1208 08:37:56.041795   12806 certs.go:167] generating certificate for "etcd-c"
I1208 08:37:56.499683   12806 certs.go:167] generating certificate for "etcd-manager-etcd-c"
...
I1208 08:37:56.753268   12806 certs.go:167] generating certificate for "etcd-c"
```

This needs to be done on all Kubernetes master nodes/etcd pods.

#### *Fix not working Prometheus monitoring after cert rotation*

Prometheus Operator relies on a predefined secret containing etcd client certificates, key and etcd CA cert. It's pre-configured [here](https://github.com/prometheus-community/helm-charts/blob/f1729dcfd2040660d4f3dcbe3b2f797415990711/charts/kube-prometheus-stack/values.yaml#L1635). However this secret is not updated automatically with the newly generated certs by the `etcd-manager` and needs to be done manually.

The easiest way for this is to ssh into one of the Kubernetes master nodes and run the following command, but first you need to delete the old secret containing the old etcd client certs:

```
$ kubectl -n monitoring delete secret etcd-certs
```
Next, recreate the secret but this time including the newly generated certificates:
```
$ kubectl -n monitoring create secret generic etcd-certs --from-file=ca.crt=/etc/kubernetes/pki/kube-apiserver/etcd-ca.crt --from-file=client.crt=/etc/kubernetes/pki/kube-apiserver/etcd-client.crt --from-file=client.key=/etc/kubernetes/pki/kube-apiserver/etcd-client.key
```
<sup>* *replace namespace and secret name accordingly*</sup>

The only thing left is to restart the Prometheus pod so that it mounts the newly created secret containing your renewed etcd client certificates.
