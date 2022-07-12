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


Get a shell to the running Container:

$ "kubectl exec -it security-context-demo -- sh"

In your shell, list the running processes:

$ "ps"

![image](https://user-images.githubusercontent.com/88305831/178474738-e1fc27ff-b151-4384-a139-55e4e7dda612.png)

In your shell, navigate to /data, and list the one directory:

$ "cd /data; ls -l"

The output shows that the /data/demo directory has group ID 2000, which is the value of fsGroup.

![image](https://user-images.githubusercontent.com/88305831/178475098-0d495e4c-0be9-4746-b989-a791550671e2.png)


In your shell, navigate to /data/demo, and create a file:

$ "cd demo; echo hello > testfile"

List the file in the /data/demo directory:

$ "ls -l"

![image](https://user-images.githubusercontent.com/88305831/178475408-29ad4046-c5b0-4d0f-b0ff-fa1744160745.png)

Run the following command:

$ "id"

The output is similar to this:

![image](https://user-images.githubusercontent.com/88305831/178475541-5e354a30-6f25-499e-ae41-5b536eabcd74.png)

From the output, you can see that gid is 3000 which is same as the runAsGroup field. If the runAsGroup was omitted, the gid would remain as 0 (root) and the process will be able to interact with files that are owned by the root(0) group and groups that have the required group permissions for the root (0) group.

Exit your shell:

$ "exit"

---

# Set the security context for a Container


To specify security settings for a Container, include the securityContext field in the Container manifest. The securityContext field is a [SecurityContext](https://kubernetes.io/docs/reference/generated/kubernetes-api/v1.24/#securitycontext-v1-core) object. Security settings that you specify for a Container apply only to the individual Container, and they override settings made at the Pod level when there is overlap. Container settings do not affect the Pod's Volumes.

Here is the configuration file for a Pod that has one Container. Both the Pod and the Container have a securityContext field:

![image](https://user-images.githubusercontent.com/88305831/178482472-41592d46-a7fa-4e93-9ce0-9ab0d02df0dc.png)


Create the Pod:

$ "kubectl apply -f security-context-2.yaml"

Verify that the Pod's Container is running:

$ "kubectl get pod security-context-demo-2"

![image](https://user-images.githubusercontent.com/88305831/178482830-7dcc0507-a7fd-41d9-afae-569fb9617629.png)

Now here we have 2 containers "sec-ctx-demo-2" and "busybox" running in the pod "security-context-demo-2"

Get a shell into the running Container "sec-ctx-demo-2".

$ "kubectl exec -it security-context-demo-2 -c sec-ctx-demo-2 -- sh"

In your shell, list the running processes:

$ "ps aux"

The output shows that the processes are running as user 2000. This is the value of runAsUser specified for the Container. It overrides the value 1000 that is specified for the Pod.

![image](https://user-images.githubusercontent.com/88305831/178483585-12df1aeb-574a-4c52-92bb-06bcf6a103ff.png)

Similarly get a shell into the running Container "busybox".

$ "kubectl exec -it security-context-demo-2 -c busybox -- sh"

In your shell, list the running processes:

$ "ps aux"

The output shows that the processes are running as user 4000. This is the value of runAsUser specified for the Container. It overrides the value 1000 that is specified for the Pod.

![image](https://user-images.githubusercontent.com/88305831/178484002-861b389a-48ea-4624-88cc-f922f083f6dc.png)

Exit your shell:

$ "exit"

---

# Set capabilities for a Container 

With [Linux capabilities](https://man7.org/linux/man-pages/man7/capabilities.7.html), you can grant certain privileges to a process without granting all the privileges of the root user. To add or remove Linux capabilities for a Container, include the capabilities field in the securityContext section of the Container manifest.

First, see what happens when you don't include a capabilities field. Here is configuration file that does not add or remove any Container capabilities:


![image](https://user-images.githubusercontent.com/88305831/178486442-21e947dd-42e4-48c4-8b2a-2f34669bfde4.png)


Create the Pod:

$ "kubectl apply -f security-context-3.yaml"

Verify that the Pod's Container is running:

$ "kubectl get pod security-context-demo-3"

![image](https://user-images.githubusercontent.com/88305831/178486656-c4ba8d1b-eff6-4a34-a9b8-8b6d6ce263d1.png)


Get a shell into the running Container:

$ "kubectl exec -it security-context-demo-3 -- sh"

In your shell, list the running processes:

$ "ps aux"

The output shows the process IDs (PIDs) for the Container:

![image](https://user-images.githubusercontent.com/88305831/178486939-1303220e-fdd8-4d83-8842-d204903aa5b0.png)

In your shell, view the status for process 1:

$ "cd /proc/1; cat status"

The output shows the capabilities bitmap for the process:

![image](https://user-images.githubusercontent.com/88305831/178487222-f03c05d6-8e56-4c52-bcaa-df63a9f4738b.png)

Make a note of the capabilities bitmap, and then exit your shell:

$ "exit"

Next, run a Container that is the same as the preceding container, except that it has additional capabilities set.

Here is the configuration file for a Pod that runs one Container. The configuration adds the CAP_NET_ADMIN and CAP_SYS_TIME capabilities:

![image](https://user-images.githubusercontent.com/88305831/178488249-5f7a305d-cad5-4c06-909b-e8905201bff0.png)


Create the Pod:

$ "kubectl apply -f security-context-4.yaml"

Get a shell into the running Container:

$ "kubectl exec -it security-context-demo-4 -- sh"

In your shell, view the capabilities for process 1:

$ "cd /proc/1; cat status"

The output shows capabilities bitmap for the process:

![image](https://user-images.githubusercontent.com/88305831/178488913-2e74a47e-8e13-405a-a15a-84c44cb53167.png)

Compare the capabilities of the two Containers:

In the capability bitmap of the first container, bits 12 and 25 are clear. In the second container, bits 12 and 25 are set. Bit 12 is CAP_NET_ADMIN, and bit 25 is CAP_SYS_TIME. See [capability.h](https://github.com/torvalds/linux/blob/master/include/uapi/linux/capability.h) for definitions of the capability constants.

Note: Linux capability constants have the form CAP_XXX. But when you list capabilities in your container manifest, you must omit the CAP_ portion of the constant. For example, to add CAP_SYS_TIME, include SYS_TIME in your list of capabilities.

---

# Assign SELinux labels to a Container
To assign SELinux labels to a Container, include the seLinuxOptions field in the securityContext section of your Pod or Container manifest. The seLinuxOptions field is an [SELinuxOptions](https://kubernetes.io/docs/reference/generated/kubernetes-api/v1.24/#selinuxoptions-v1-core) object. Here's an example that applies an SELinux level:

![image](https://user-images.githubusercontent.com/88305831/178490197-bfe8f6c4-c2da-497e-8c4f-0bdbdbdf7bba.png)

Note: To assign SELinux labels, the SELinux security module must be loaded on the host operating system.

**Discussion**

The security context for a Pod applies to the Pod's Containers and also to the Pod's Volumes when applicable. Specifically fsGroup and seLinuxOptions are applied to Volumes as follows:

fsGroup: Volumes that support ownership management are modified to be owned and writable by the GID specified in fsGroup. See the [Ownership Management design document](https://github.com/kubernetes/design-proposals-archive/blob/main/storage/volume-ownership-management.md) for more details.

seLinuxOptions: Volumes that support SELinux labeling are relabeled to be accessible by the label specified under seLinuxOptions. Usually you only need to set the level section. This sets the [Multi-Category Security (MCS)](https://selinuxproject.org/page/NB_MLS) label given to all Containers in the Pod as well as the Volumes.

Warning: After you specify an MCS label for a Pod, all Pods with the same label can access the Volume. If you need inter-Pod protection, you must assign a unique MCS label to each Pod.

**Clean up**

Delete the Pod:

$ "kubectl delete pod security-context-demo"
$ "kubectl delete pod security-context-demo-2"
$ "kubectl delete pod security-context-demo-3"
$ "kubectl delete pod security-context-demo-4"







