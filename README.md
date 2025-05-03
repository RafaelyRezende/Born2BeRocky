# Born-4-2beroot

<br>

### System Administrator Diary

This is a guide through the essential concepts and steps to completing 42 project Born2beroot with Rocky Linux.

<a href="https://rockylinux.org/">Rocky Linux</a> is a distribution stems from CentOS since its discontinued development. The original co-founder of <a href="https://en.wikipedia.org/wiki/CentOS">CentOS</a> took on the responsability of carrying on the initial goal of a community-driven, enterprise level operating system for development. The project is now hosted by <a href="https://www.resf.org/about">Rocky Enterprise Software Foundation</a>.

Rocky established itself as a downstream build of its upstream vendor <a href="https://en.wikipedia.org/wiki/Red_Hat_Enterprise_Linux">Red Hat Enterprise Linux</a>. 

### Rocky OS Install

The installation ISO can be found at the Rocky OS website under the section of default images. Download Rocky Linux 9.5 minimal ISO (the checksum can be used to verify the integrity of the installation).

On the virtual machine software of your choice (this guide will use Oracle VirtualBox), Figure 0, create a new virtual machine, choose the name, select the directory to save the VM files and select the directory with the Rocky ISO image. Check the box fot the 'Skip Unattended Installation'.

<p align="center">
  <img src="https://github.com/RafaelyRezende/Born-4-2beroot/blob/main/rocky_guide/02-VMsetup.png" width=50% height=50% alt="Create Virtual Machine VirtualBox menu">
</p>
<p align="center">
    <em>Figure 0.</em>
</p>

In the 'Hardware' section (Figure 1), select the amount of base memory for the virtual machine and the amount of processors you want to use. 

<p align="center">
  <img src="https://github.com/RafaelyRezende/Born-4-2beroot/blob/main/rocky_guide/03-VMsetup.png" width=50% height=50% alt="VirtualBox hardware menu">
</p>
<p align="center">
    <em>Figure 1.</em>
</p>

Next in the 'Virtual Hard Disk' shown in Figure 2, create a virtual hard disk with the amount specified in the subject <strong>(this size will change in case you choose to make the bonus)</strong>. Take some time to do the final check of the specifications for the VM and finish the creation.

<p align="center">
  <img src="https://github.com/RafaelyRezende/Born-4-2beroot/blob/main/rocky_guide/04-VMsetup.png" width=50% height=50% alt="VirtualBox Virtual Hard Disk menu">
</p>
<p align="center">
    <em>Figure 2.</em>
</p>

### Disk Partition

#### Partitioning Scheme Overview

The partition scheme, as per the bonus section, must have one primary partition and an extended partition for the logical volume groups, Figure 3.

<p align="center">
  <img src="https://github.com/RafaelyRezende/Born-4-2beroot/blob/main/rocky_guide/partionsqueme.png" width=50% height=50% alt="Partition Scheme">
</p>
<p align="center">
    <em>Figure 3.</em>
</p>

The virtual machine has a set amount of primary memory (RAM) and a set amount of secondary memory (hard disk or SSD). The objective is slice the available secondary memory into different sectors that will compartimentalize different parts of the operating system. In order to achieve this goal, first the partitions must be create following a certain partition table, the extended partition must be encrypted and inside it logical volume groups are create to support different directories of the linux file system.

The partitioning scheme used is the legacy Master Boot Record (MBR). This type provides wide compatibility with older systems and has a simple structure to be worked on. MBR has a set number of primary partitions that can be created, no more than 4, and can only support 2 TiB of size disk which will be more than enough for the pourposes of this project. The up to date, modern standard of partitioning scheme is GUID Partition Table (GPT) with almost every single specification having an upgrade compared to its predecessor MBR. GPT has practically an unlimited amount of partitions that can be created, it is realiable given its system of redundancy checks, it has compatibility with mordern boot firmware such as UEFI and it can manage larger systems with sizes bigger than 2 TiB.

#### File Systems and Mount Point Overview

After the disk has been properly partitioned, the system is ready to have each partition formatted with a file system. 

### SELinux

Security-Enhanced Linux is a security layer built mixed with the kernel in some GNU/Linux distributions for, you guessed, enhanced security over sensitive data and processes. It was developed in a joint colaboration between linux developers and the National Security Agency (NSA). The feature allows administrators to have advanced and fine granied control over the access and permissions of the system.

It uses security policies, a set of rules for deciding what can and can not be accessed, to enforce the entry allowed by a policy. In a situation where a subject, term used to categorized applications or processess, makes a request to access an object, for example a file or a directory, SELinux guarantee such subject has the permission to modify, read or write such object by checking the <a href="https://docs.redhat.com/en/documentation/red_hat_enterprise_linux/7/html/selinux_users_and_administrators_guide/sect-security-enhanced_linux-introduction-selinux_architecture">access vector cache</a> (AVC). The said permissions context are loaded into a cache at boot time.

SELinux can run in three different modes of operation. The default is the enforcing mode, the recommended mode, where the policies apllied follow the labels loaded in cache. The <code>setenforce</code>

### Create and manage new users and groups

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

## Partitions

After setting up the virtual machine to use the Rocky 9.5 minimal iso and the memory size given to the new virtual machine, enter the text mode of instalation. To do so press <italic>Tab</italic> and type <code>inst.text</code>.

This will lead you to a menu, choose the text
