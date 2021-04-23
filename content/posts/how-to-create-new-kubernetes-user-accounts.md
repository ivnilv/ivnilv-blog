---
title: "How to Create New Kubernetes User Accounts"
date: 2021-03-15T00:26:43+02:00
draft: false
cover: "/images/k8s.png"
tags:
- kubernetes
---

### Generate new certificate

First, we have to generate a private key:

{{< highlight bash >}}
$ openssl genrsa -out ivnilv.pem
{{< / highlight >}}

and a certificate signing request:

{{< highlight bash >}}
$ openssl req -new -key ivnilv.pem -out ivnilv.csr -subj "/CN=ivnilv"
{{< / highlight >}}


The common name of the certificate is important, since it defines the username of the new user.

### Signing the certificate

The signing request needs to be base64 encoded , before submitting to the Kubernetes API. You can easily encode it using:

{{< highlight bash >}}
$ cat ivnilv.csr | base64 | tr -d '\n'
{{< / highlight >}}

The signing request has to be signed by Kubernetes CA now. 

{{< highlight bash >}}
apiVersion: certificates.k8s.io/v1beta1
kind: CertificateSigningRequest
metadata:
  name: user-request-ivnilv
spec:
  groups:
  - system:authenticated
  request: LS0tLS1CRUdJTiBDRVJUSUZJQ0FURSBSRVFVRVNULS0tLS0KTUlJQ1ZUQ0NBVDBDQVFBd0VERU9NQXdHQTFVRUF3d0ZhbUZyZFdJd2dnRWlNQTBHQ1NxR1NJYjNEUUVCQVFVQQpBNElCRHdBd2dnRUtBb0lCQVFDNGxQMTNVZHlBTDZ1QUttTVROeDU5RUtlTE14VkNWSTJVWUhWL2hvYjdpelBGCldRSmxaY1dEWllnL1dIVTJoMFJ3TGNkeWpaVTVHcTFPdS9IMjQxSWpBL3dTMXJSWnc1c0ZrNG8zYWdBZlJ0QngKbzZDc3czS0Q3eEpvdEtpRURzOXFqbUJPNzFwU1kzK3BqdHBZbUpxWmZ5cmV2Y094Wm12NXpSU2NaWUZkb1ZMdwptb0VxeUx0a1hDNzVpeXBGTFg2WjJwK25POFE2NHFtU2VZWHRjNFZBbjZhbjdlUFJFU3k5S09NUURZTHRUZENBClBaTVpDeXV0WnBXQnFqMXZlV3NkaWxweUlyaXdRYWtNQ2tBdURHbUxGbzJhbFk3REV4Z0t0ay8rZlZSaEkyY1YKRUJzVUptaVBuZ1NrMXhKRzZjWjdUZE12UXYxSnIyaGNsc1NDY1IySkFnTUJBQUdnQURBTkJna3Foa2lHOXcwQgpBUXNGQUFPQ0FRRUFiL1BBcVp0eUovUHA2MWtXcG54azNDek9SNTdhT09JSHU3YkVUN1lmcE9GazFSN0JkV3hWClU1RXUrK3pTNVZmQm9XaVNNekV1eENrVHZsNU1CVGxDL3NvQWprT2YzQTI2aUFpeFJibksrUWw0ZGt1MGJQTisKT2syZ3pqTXZyM1hsT1ExZVZnQytjTVZzeXJpZENUMzRwcHVPNm9RTkZnVk1GblZ3bHozdjJnZkVDK3c1RnVndgpOVXcxT21td0c1RTZKN0VpL2ZSTk1scnUyTzZaUlQvMGYyWWpnSUhwT3RONTJYSDhhc3cvMjhMOWt1c1J6aHR5CnFLa0xxWGhkYW4xSDRsR1E2VUwxd1V4UU8zUWhyRnUrbkhFRTV3SVdDNEFra0N4K1NPQ3RpalNmV2Vhb3dkTzcKRG8zaWJvWGNwTzVSUDYvYUhEWE9kV0ZXc2EvQituVG9Vdz09Ci0tLS0tRU5EIENFUlRJRklDQVRFIFJFUVVFU1QtLS0tLQo=
  usages:
  - digital signature
  - key encipherment
  - client auth
{{< / highlight >}}

and use it to create the resource:

{{< highlight bash >}}
$ kubectl create -f ivnilv.yaml 
certificatesigningrequest.certificates.k8s.io/user-request-ivnilv created
{{< / highlight >}}

The YAML file has to contain base64 encoded version of your signing request (the .csr file). 
But Kubernetes won't sign the new user certificate just like that. They will wait for approval that this certificate can be really signed. That can be done again using kubectl:

{{< highlight bash >}}
$ kubectl certificate approve user-request-ivnilv
certificatesigningrequest.certificates.k8s.io/user-request-ivnilv approved

$ kubectl get csr
NAME                     AGE       REQUESTOR   CONDITION
user-request-ivnilv   1m        admin       Approved,Issued
{{< / highlight >}}

Now the certificate should be signed. You can download the new signed public key from the csr resource:

{{< highlight bash >}}
kubectl get csr user-request-ivnilv -o jsonpath='{.status.certificate}' | base64 -d > ivnilv.crt
{{< / highlight >}}

### Create new user config file

Now that we have all the files needed for compiling the `kubeconfig` for our new user, we can issue the following commands (replacing with your own values) to generate a new config file under `~/.kube/`:

{{< highlight bash >}}
kubectl --kubeconfig ~/.kube/config-ivnilv config set-cluster preprod --insecure-skip-tls-verify=true --server=https://KUBERNETES-API-ADDRESS
kubectl --kubeconfig ~/.kube/config-ivnilv config set-credentials ivnilv --client-certificate=ivnilv.crt --client-key=ivnilv.pem --embed-certs=true
kubectl --kubeconfig ~/.kube/config-ivnilv config set-context default --cluster=preprod --user=ivnilv
kubectl --kubeconfig ~/.kube/config-ivnilv config use-context default
{{< / highlight >}}

Let's test if the new `kubeconfig` we generated worked fine:

{{< highlight bash >}}
$ kubectl --kubeconfig ~/.kube/config-ivnilv get pods
No resources found.
Error from server (Forbidden): pods is forbidden: User "ivnilv" cannot list pods in the namespace "default"
{{< / highlight >}}

If you get this forbidden error, then user was created successfully, but we still doesn't have any access rights. 

At this point you are done with this guide and you should continue on the next one for assigning appropriate roles and access rights to the new created user.