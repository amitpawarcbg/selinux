# selinux
**Security Context for a Pod or Container**

A security context defines privilege and access control settings for a Pod or Container. Security context settings include, but are not limited to:

* *Discretionary Access Control*: Permission to access an object, like a file, is based on [user ID (UID) and group ID (GID)](https://wiki.archlinux.org/title/users_and_groups).

* *[Security Enhanced Linux (SELinux)](https://en.wikipedia.org/wiki/Security-Enhanced_Linux)*: Objects are assigned security labels.

* Running as privileged or unprivileged.

* *[Linux Capabilities](https://linux-audit.com/linux-capabilities-hardening-linux-binaries-by-removing-setuid/)*: Give a process some privileges, but not all the privileges of the root user.

* *[AppArmor](https://kubernetes.io/docs/tutorials/security/apparmor/)*: Use program profiles to restrict the capabilities of individual programs.

* *[Seccomp](https://kubernetes.io/docs/tutorials/security/seccomp/)*: Filter a process's system calls.

* *allowPrivilegeEscalation*: Controls whether a process can gain more privileges than its parent process. This bool directly controls whether the no_new_privs flag gets set on the container process. allowPrivilegeEscalation is always true when the container:

1. is run as privileged, or
2. has CAP_SYS_ADMIN

* *readOnlyRootFilesystem*: Mounts the container's root filesystem as read-only.

---

# Let's get through some concepts for selinux - Security Context for a Pod or Container.

* Linux Syscalls

![image](https://user-images.githubusercontent.com/88305831/178442709-0729c3ae-4e2c-4a9a-a876-0452e254ffa0.png)

* Security context for Pod or containers

![image](https://user-images.githubusercontent.com/88305831/178443726-130d257b-871d-4e63-80b1-ea10cdca0b19.png)


![image](https://user-images.githubusercontent.com/88305831/178444257-e3c00126-52a3-418f-b571-2b0fd45d7e95.png)


To specify security settings for a Pod, include the securityContext field in the Pod specification. The securityContext field is a [PodSecurityContext](https://kubernetes.io/docs/reference/generated/kubernetes-api/v1.24/#podsecuritycontext-v1-core) object. The security settings that you specify for a Pod apply to all Containers in the Pod. Here is a configuration file for a Pod that has a securityContext and an emptyDir volume:

![image](https://user-images.githubusercontent.com/88305831/178446082-592945df-b505-42a7-9bc4-c5a125149d2f.png)

In the configuration file, the runAsUser field specifies that for any Containers in the Pod, all processes run with user ID 1000. The runAsGroup field specifies the primary group ID of 3000 for all processes within any containers of the Pod. If this field is omitted, the primary group ID of the containers will be root(0). Any files created will also be owned by user 1000 and group 3000 when runAsGroup is specified. Since fsGroup field is specified, all processes of the container are also part of the supplementary group ID 2000. The owner for volume /data/demo and any files created in that volume will be Group ID 2000.

Create the Pod:

$ "kubectl apply -f security-context.yaml"

Verify that the Pod's Container is running:

$ "kubectl get pod security-context-demo"

![image](https://user-images.githubusercontent.com/88305831/178460547-ca3a2543-72da-4430-87c5-61aa9b68487f.png)






