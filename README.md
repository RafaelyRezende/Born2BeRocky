# Born-4-2beroot

<br>

### System Administrator Diary

This is a guide through the essential concepts and steps to completing 42 project Born2beroot with Rocky Linux.

<a href="https://rockylinux.org/">Rocky Linux</a> is a distribution that stems from CentOS since its discontinued development. The original co-founder of <a href="https://en.wikipedia.org/wiki/CentOS">CentOS</a> took on the responsability of carrying on the initial goal of a community-driven, enterprise level operating system for development. The project is now hosted by <a href="https://www.resf.org/about">Rocky Enterprise Software Foundation</a>.

Rocky established itself as a downstream build of its upstream vendor <a href="https://en.wikipedia.org/wiki/Red_Hat_Enterprise_Linux">Red Hat Enterprise Linux</a>.

### SELinux

Security-Enhanced Linux is a security layer built mixed with the kernel in some GNU/Linux distributions for, you guessed, enhanced security over sensitive data and processes. It was developed in a joint colaboration between linux developers and the National Security Agency (NSA). The feature allows administrators to have advanced and fine granied control over the access and permissions of the system.

It uses security policies, a set of rules for deciding what can and can not be accessed, to enforce the entry allowed by a policy. In a situation where a subject, term used to categorized applications or processess, makes a request to access an object, for example a file or a directory, SELinux guarantee such subject has the permission to modify, read or write such object by checking the <a href="https://docs.redhat.com/en/documentation/red_hat_enterprise_linux/7/html/selinux_users_and_administrators_guide/sect-security-enhanced_linux-introduction-selinux_architecture">access vector cache</a> (AVC). The said permissions context are loaded into a cache at boot time.

SELinux can run in three different modes of operation. The default is the enforcing mode, the recommended mode, where the policies apllied follow the labels loaded in cache. The <code>setenforce</code>

## Create and manage new users and groups

<code>useradd -u 4242 -d /home/<username> -m <username></code>

Some useful flags are the -u flag to set an specific user ID number, the -d flag sets the path of the home directory for the new user and -m creates the home directory. More flags can be view with the usual --help flag.

<code>passwd <username></code>

The command above is used to set a user's password. Follow the prompt instructions, later I will show how to customize the rules for strong password policy and add custom messages.

You can verify the existing groups in your server with the following command <code>cat /etc/group</code>.

<code>usermod -a -G <groupname> <username></code>

The command above is used to add an user to a group. The -a flag stands for append and the -G flag tells the usermod command that the you want to edit groups.

<code>groupadd <groupname></code>

The above command is used to create a new group.

## Manage sudo access commands

<code>visudo</code> command opens the /etc/sudoers file in our system where you can edit for specific permissions and configuration.

## Hostname

<code>hostnamectl <newhostname></code>
