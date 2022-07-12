# selinux
**Security Context for a Pod or Container**

A security context defines privilege and access control settings for a Pod or Container. Security context settings include, but are not limited to:

* *Discretionary Access Control*: Permission to access an object, like a file, is based on user ID (UID) and group ID (GID).

* *Security Enhanced Linux (SELinux)*: Objects are assigned security labels.

* Running as privileged or unprivileged.

* *Linux Capabilities*: Give a process some privileges, but not all the privileges of the root user.

* *AppArmor*: Use program profiles to restrict the capabilities of individual programs.

* *Seccomp*: Filter a process's system calls.

* *allowPrivilegeEscalation*: Controls whether a process can gain more privileges than its parent process. This bool directly controls whether the no_new_privs flag gets set on the container process. allowPrivilegeEscalation is always true when the container:

1. is run as privileged, or
2. has CAP_SYS_ADMIN

* *readOnlyRootFilesystem*: Mounts the container's root filesystem as read-only.

---

# Let's get through some concepts for selinux - Security Context for a Pod or Container.
