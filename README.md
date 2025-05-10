# Born 2 Be Root

<br>

## Table of Contents

- [Prelude](#prelude)
- [Rocky OS Install](#rocky-os-install)
- [Disk Partition](#disk-partition)
  - [Partitioning Scheme Overview](#partitionin-scheme-overview)
  - [Filesystems and Mount Point Overview](#filesystems-and-mount-point-overview)
  - [Logical Volume Management](#logical-volume-management)
  - [Disk Setup](#disk-setup)
- [Inside The Machine](#inside-the-machine)
  - [Secure Shell](#secure-shell)
  - [Firewalld](#firewalld)
  - [SELinux](#selinux)
  - [SSH Setup](#ssh-setup)
  - [Hostname](#hostname)
  - [Users and Groups](#users-and-groups)
  - [Secure Password Policy](#secure-password-policy)
  - [Sudo Configuration](#sudo-configuration)
  - [Monitor & Verify](#monitor-&-verify)
  - [Mandatory Check](#mandatory-check)
- [Bonus Services](#bonus-services)
  - [Lighttpd](#lighttpd)
  - [MariaDB](#mariadb)
  - [PHP](#php)
  - [WordPress](#wordpress)
- [Full Stack Setup](#full-stack-setup)
  - [Extra Service](#extra-service)
- [Wrap up](#wrap-up)

## Prelude

Before you dive deep into this project, I want to say a few words about the people that helped and collaborated on this project...

When all of us got together to decide which distribution we would pick for this project, I was the only one that consider it, out of pure stupidity and ignorance. I did not know what the project was about and did not know the amount of work it would take for it to be completed. I was immensely lucky to have <strong>Alex Barbosa Felix Da Silva</strong>, <strong>Júlio César Santos Souza</strong> and <strong>Nicholas Saraiva Arruda Serafim</strong>, joining together with me to tackle the project. I would not even got to the installation screen without their help and insights. In true 42 spirit and collaboration, I can say for sure that we all learned greatly from the whole experience and I wish for you reading this all the same (if possible even pass it on).

I salute my fellow friends, more to us like these I wish.

---

Points of contact:

Alex Barbosa Felix Da Silva -> alebarbo@student.42porto.com

<a href="https://github.com/JulioSouza09">Júlio César Santos Souza</a> -> jcesar-s@student.42porto.com

<a href="https://github.com/CafeComLag">Nicholas Saraiva Arruda Serafim</a> -> nsaraiva@student.42porto.com

---

## Rocky OS Install

<a href="https://rockylinux.org/">Rocky Linux</a> is a distribution which stems from CentOS since its discontinued development. The original co-founder of <a href="https://en.wikipedia.org/wiki/CentOS">CentOS</a> took on the responsability of carrying on the initial goal of a community-driven, enterprise level operating system for development. The project is now hosted by <a href="https://www.resf.org/about">Rocky Enterprise Software Foundation</a>.

Rocky established itself as a downstream build of its upstream vendor <a href="https://en.wikipedia.org/wiki/Red_Hat_Enterprise_Linux">Red Hat Enterprise Linux</a>.

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

## Disk Partition

### Partitioning Scheme Overview

The partition scheme, as per the bonus section, must have one primary partition and an extended partition for the logical volume groups, Figure 3.

<p align="center">
  <img src="https://github.com/RafaelyRezende/Born-4-2beroot/blob/main/rocky_guide/partionsqueme.png" width=50% height=50% alt="Partition Scheme">
</p>
<p align="center">
    <em>Figure 3.</em>
</p>

The virtual machine has a set amount of primary memory (RAM) and a set amount of secondary memory (hard disk or SSD). The objective is slice the available secondary memory into different sectors that will compartimentalize different parts of the operating system. In order to achieve this goal, first the partitions must be create following a certain partition table, the extended partition must be encrypted and inside it, logical volume groups are created to support different directories of the linux filesystem.

The partitioning scheme layout standard used is the legacy Master Boot Record (MBR). This type provides wide compatibility with older systems and has a simple structure to be worked on. MBR has a set number of primary partitions that can be created, no more than 4, and can only support 2 TiB of size disk which will be more than enough for the pourposes of this project. The up to date, modern standard of partitioning scheme is GUID Partition Table (GPT) with almost every single specification having an upgrade compared to its predecessor MBR. GPT has practically an unlimited amount of partitions that can be created, it is realiable given its system of redundancy checks, it has compatibility with mordern boot firmware such as UEFI and it can manage larger systems with sizes bigger than 2 TiB. 

More information about disk partititons in <a href="https://docs.fedoraproject.org/en-US/fedora/f36/install-guide/appendixes/Disk_Partitions/">here</a> and <a href="https://en.wikipedia.org/wiki/Disk_partitioning">there</a>.

### Filesystems and Mount Point Overview

After the disk has been properly partitioned, the system is ready to have each partition formatted with a filesystem. Each pool of memory now has to be assigned a filesystem format and a mount point. Linux suppports a wide range of filesystems, each with its own particularities, characteristics and performance according to given task.

Filesystems simply structures the way data is stored, organized, accessed and managed throughout the operating system. It adds redunduncy in the form of journals or logs for the case of sudden crashes or system corruption. It keeps track of the area in which data must be stored and can be used. Also, it implements checksums to verify integrity of the system and file modifications.

This project uses the ext4 filesystem for its stabililty and performance which is enough in this case. The ext4 is flexible which make it suitable for a variety of workloads and file sizes. The current project does not require the management of large storage units, scalability is not the main goal, so the ext4 is the right fit.

Useful content around this topic and other types of filesystems <a href="https://archive.kernel.org/oldwiki/ext4.wiki.kernel.org/index.php/Ext4_Howto.html">here</a> and <a href="https://en.wikipedia.org/wiki/Ext4">there</a>.

### Logical Volume Management

Logical volume management is a device mapper framework that abstracts the physical storage devices on a linux system. Physical memory storage can now be virtualized in virtual block devices, this make possible for the logical volumes to absorb new physical devices to enlarge the systems size or shrink it dynamically.

The layered architecture of the LVM is composed by the physical volume which is the base layer, normally an entire partition. The volume group is the central pool of storage composed of one or more physical volumes, this layer acts a container for the physical volumes. The logical volume is the abstraction created for the operating system to use. The logical volume is a standard block device in the perspective of the OS where mount points can be assigned and formatted with specific file system.

More information on <a href="https://docs.redhat.com/en/documentation/red_hat_enterprise_linux/9/html/configuring_and_managing_logical_volumes/overview-of-logical-volume-management_configuring-and-managing-logical-volumes#lvm-architecture_overview-of-logical-volume-management">here</a> and <a href="https://en.wikipedia.org/wiki/Device_mapper">there</a>.

### Disk Setup

At the start up installation menu (Figure 4), press <i>TAB</i>, type <code>inst.text</code> and press enter. Next, select the text mode and go into the anaconda prompt by pressing <i>alt+tab</i>. 

<p align="center">
  <img src="https://github.com/RafaelyRezende/Born-4-2beroot/blob/main/rocky_guide/rocky_install01.png" width=50% height=50% alt="Installation menu">
</p>
<p align="center">
    <em>Figure 4.</em>
</p>

At the anaconda prompt, use the fdisk command utility to write on the /dev/sda disk (Figure 5).

<p align="center">
  <img src="https://github.com/RafaelyRezende/Born-4-2beroot/blob/main/rocky_guide/rocky_install04.png" width=50% height=50% alt="Anaconda prompt">
</p>
<p align="center">
    <em>Figure 5.</em>
</p>

Inside the fdisk program, type m for the help menu. Afterwards, type n to create a new partition, select it as a primary partition and set the last sector to be +500M as shown in Figure 6. Leave the <i>Partition number</i> as the default, as well as, the first sector. 

<p align="center">
  <img src="https://github.com/RafaelyRezende/Born-4-2beroot/blob/main/rocky_guide/rocky_install05.png" width=50% height=50% alt="Create primary partition">
</p>
<p align="center">
    <em>Figure 6.</em>
</p>

After creating the primary partition where the /boot mount point will live, add another partition, only this time select it as an extended partition.The fdisk program creates a second partition named sda2 of type 'Extended' with all the remaining space available on disk. Create yet another partition with the n command, with all the space allocated to the extended partition sda2, fdisk will add a logical partition, leave again all the fields as the default. Type w to update the partition table. Follow the mentioned steps and check if the process is similar to Figure 7.

<p align="center">
  <img src="https://github.com/RafaelyRezende/Born-4-2beroot/blob/main/rocky_guide/rocky_install07.png" width=50% height=50% alt="Installation menu">
</p>
<p align="center">
    <em>Figure 7.</em>
</p>

The sda5 partition needs to be encrypted and for this action use the following command on the anaconda prompt:

<code>cryptsetup -y -v --type luks1 luksFormat /dev/sda5</code>

The command above basically sets the desired type of encryption, luks1 or luks2, on top of the of the sda5 partition. By encrypting the entire partition, the part of the system where potentially critical information is stored will be safe.

With the partition encrypted, in order to add the logical volume manager (LVM) groups it is necessary to open the partition first. This can be achieved with the following command:

<code>cryptsetup open /dev/sda5 sda5_crypt</code>

Now, the extended partition can be managed to have any logical groups needed. First, create the physical volume in which the logical volumes will reside with the command:

<code>pvcreate /dev/mapper/sda5_crypt</code>. 

Afterwards, create the volume group which the logical volumes will be a part of on top of the newly created physical volume mapper. Use the command:

<code>vgcreate LVMGroup /dev/mapper/sda5_crypt</code>.

Finally, create all the necessary logical volumes that belong to the LVMGroup. Use the <i>lvcreate</i> command to achieve this goal. The size can be set with the '-L' flag, and the name of the logical volume with the '-n' flag. For example:

<code>lvcreate -L 10G -n root LVMGroup</code>

Repeat this step for all the logical volumes, <strong>do this in the order as per the subject</strong>. Check if the partition is similar to Figure 8 with the <code>lsblk</code> command.

<p align="center">
  <img src="https://github.com/RafaelyRezende/Born-4-2beroot/blob/main/rocky_guide/rocky_install09.png" width=50% height=50% alt="Final partition table">
</p>
<p align="center">
    <em>Figure 8.</em>
</p>

When all these steps are completed, type <strong>reboot</strong> in the command prompt and enter the guided installation wizard. In the 'Installation Summary' menu, under 'System' select the 'Installation Destination'. Select the checkbox of the manual installation and hit 'Done', this will redirect to the 'Manual Partitioning' menu. If you did every step the correct way, there will be a 'Unknown' header in the 'New Rocky Linux 9.5 Installation' as depicted in Figure 9. Open the header to find the sda1 and sda5 encrypted partition, open the sda5 partition with the password.

<p align="center">
  <img src="https://github.com/RafaelyRezende/Born-4-2beroot/blob/main/rocky_guide/rocky_install10.png" width=50% height=50% alt="Graphical installation">
</p>
<p align="center">
    <em>Figure 9.</em>
</p>

Now all the logical volumes and primary partition can be reformated and mounted properly. Select a logical volume, check the 'Reformat' checkbox, edit the 'Mount Point' field and select a filesystem type in the 'File System' field. Press the 'Update Settings' to update the information and repeat for all the logical volumes. The figure below is an example of what to expect:

<p align="center">
  <img src="https://github.com/RafaelyRezende/Born-4-2beroot/blob/main/rocky_guide/rocky_install11.png" width=50% height=50% alt="Installation menu">
</p>
<p align="center">
    <em>Figure 10.</em>
</p>

<strong>NOTE</strong>: the swap volume has a unique filesystem type.

Create a user <strong>without</strong> administrative powers (this will be set up inside the server) and create a root password for the super user. Begin installation.

## Inside The Machine

After the final reboot of the installation, decrypt the disk and enter the user login and password to access the server. At this stage, a serie of actions must be completed to make the server secure and operational with different types of services. A list of the objectives is shown below:

<table border="5" align="center">
 <tr>
    <td>1. Set up SSH service</td>
    <td>6. Create bash script</td>
 </tr>
 <tr>
    <td>2. Change hostname</td>
    <td>7. Set up lighttpd service</td>
 </tr>
 <tr>
    <td>3. Create groups and users</td>
    <td>8. Set up mariadb service</td>
 </tr>
   <tr>
    <td>4. Implement secure password policy</td>
    <td>9. Set up WordPress website</td>
 </tr>
   <tr>
    <td>5. Set up sudo rules</td>
    <td>10. Set up additional service</td>
 </tr>
</table>

At this point, the virtual machine has a full operating system installed and operational. The VM harness the processing power, memory, disk and other physical resources from the host hardware. "An entire OS-level virtualization enables multiple isolated and secure cirtualized servers to run using only a single physical server" (<a href="https://en.wikipedia.org/wiki/Virtual_machine">source here</a>). Virtual Machine can be defined as:

>  "An efficient, isolated duplicate of a real computer machine." - Gerald J. Popek & Robert P. Goldberg.

Before the set up and configuration of the server, some topics will be introduced for better comprehension and utility of the project.

___

### Secure Shell

Secure shell (SSH) is a cryptographic communication protocol over insecure mediums. It enable secure remote access to computers and servers, over the internet. This is the service that allow developers to work from home, administer networks and servers from a distance in some third world beach around the world.

The primary goal of SSH is to secure remote login and command execution to a server or network which enable the capacity to manage, transfer and administer services inside the said network/server. This program came to replace the previoius client-server application protocols, such as <a href="https://en.wikipedia.org/wiki/Telnet">Telnet</a>, <a href="https://en.wikipedia.org/wiki/Berkeley_r-commands">rlogin</a> and <a href="https://en.wikipedia.org/wiki/Remote_Shell">rsh</a>. The SSH protocol at its inception in 1995 gain rapid adoption by the community and now stands as the golden standard of secure system administration.

The OpenSSH, a free open-source software (FOSS) implementation, is pre-installed on the majority of Linux distributions including Rocky. The service must be running, normally as server side <a href="https://en.wikipedia.org/wiki/Daemon_(computing)">daemon</a>, which makes it possible for a client (e.g. user's local machine) innitiate a connection over <a href="">Transmission Control Protocol</a> (TCP) to the server on a specific port. The default port for a SSH service is the port 22.

The architecture of SSH is organized as a layered architecture. This design provides modularity, flexibily and clear separation of concerns which contribute to the maintance, robustness and security. There are three main layers that build on top of each other, the first one is the <a href="https://www.rfc-editor.org/rfc/rfc4253">transport layer protocol</a>, the second is the user authentication protocol and the Connection Protocol. 

The first layer provides the low level implementation of communication protocol that provides strong encryption, cryptographic host authentication and integrity protection. The second is used to process client-side requests, by managing password authentication, public key authentication and other forms. And, finally, the <a href="https://www.rfc-editor.org/rfc/rfc4254#section-1">connection protocol</a>, as defined by the Internet Engineering Task Force reference, establishes "interactive login sessions, remote execution of commands, forwarded TCP/IP connections, and forwarded X11 connections. All of these channels are multiplexed into a single encrypted tunnel".

### Firewalld

A firewall is a program that monitors and administer communications send and recieved by a system. It is configured to follow certain rules for the process and flow of communication. This protects the system from unwanted traffic from outside actors and minimizes attack vectors via ports, for example.

The case for <a href="https://firewalld.org/documentation/concepts.html">firewalld</a> comes from its predecessor, iptables. The goal of firewalld is to simplify the complexities of iptables in implementing firewall rules and policy. Firewalld offers a dynamic and user-friendly method to manage firewall rules by using abstractions like zones, binning services which allows a fine grained configuration, and maintain seperate configurations for runtime processes and the rules saved to disk (permanent).

Firewalld comes pre-installed and enabled by default on RHEL distribution and Rocky. The default zone for network interfaces is set to public, normally used in public networks where other computers of the network are not trustworthy. The utility command to manage the firewalld policies and rules is the <a href="https://docs.redhat.com/en/documentation/red_hat_enterprise_linux/9/html/configuring_firewalls_and_packet_filters/using-and-configuring-firewalld_firewall-packet-filters#using-and-configuring-firewalld_firewall-packet-filters">firewall-cmd</a>.

### SELinux

Security-Enhanced Linux is a security layer built mixed with the kernel in some GNU/Linux distributions for, you guessed, enhanced security over sensitive data and processes. It was developed in a joint colaboration between linux developers and the National Security Agency (NSA). The feature allows administrators to have advanced and fine granied control over the access and permissions of the system.

It uses <a href="https://en.wikipedia.org/wiki/Mandatory_access_control">Mandatory Access Control</a> (MAC) security policies, a set of rules for deciding what can and can not be accessed, to enforce the policy of entry for allowed users, file/directory permissions, services connectivity and more. In a situation where a subject, term used to categorized applications or processess, makes a request to access an object, for example a file or a directory, SELinux guarantee such subject has the permission to modify, read or write such object by checking the <a href="https://docs.redhat.com/en/documentation/red_hat_enterprise_linux/7/html/selinux_users_and_administrators_guide/sect-security-enhanced_linux-introduction-selinux_architecture">access vector cache</a> (AVC). The said permissions context are loaded into a cache at boot time.

SELinux can run in three different modes of operation. The default is the enforcing mode, the recommended mode, where the policies apllied follow the labels loaded in cache. Verify the status of SELinux with the command:

<code>getenforce</code>

If the status of SELinux needs to be modified temporarily the following command can be used to set it to different modes of operation, more on <a href="https://www.thegeeksearch.com/how-to-use-setenforce-command-to-change-selinux-modes/">here</a>:

<code>setenforce</code>

___

### SSH Setup

First step in setting up the SSH service on port 4242 is to download the tools to manage the SELinux rules and policy. Start by installing the selinux-policy-targeted package which provides the semanage command:

<code>dnf install selinux-policy-targeted</code>

<code>dnf install policycoreutils-python-utils</code>

With these utilities installed, now it is possible to add port 4242 to the right type and context. Run the following command:

<code>semanage port -a -t ssh_port_t -p tcp 4242</code>

List the ports types in the SELinux type and verify that port 4242 was correctly added. The output of the following command must be similar to Figure 11.

<code>semanage port -l | grep ssh</code>

<p align="center">
  <img src="https://github.com/RafaelyRezende/Born-4-2beroot/blob/main/rocky_guide/rocky_install16.png" width=50% height=50% alt="semanage commands">
</p>
<p align="center">
    <em>Figure 11: semanage commands.</em>
</p>

Generate a SSH key pair for the VM. This is the standard process of creating a public-private key, use the <code>ssh-keygen</code> command and follow the instructions prompted by the program. The SSH key allow the server to stablish a remote secure connection with asymmetric cryptography. 

<code>ssh-keygen -t rsa</code>

Navigate to the <i>/etc/ssh/sshd_config</i> file to edit the default configuration of the ssh service. Read through the file and find the line which contain the port number, if you have not edited this file before it will be set to the default port.

Also in this file, change the permission for root login by setting it to 'no'. The final document after modification is presented in Figure 12 below:

<p align="center">
  <img src="https://github.com/RafaelyRezende/Born-4-2beroot/blob/main/rocky_guide/rocky_install17.png" width=50% height=50% alt="ssh config file.">
</p>
<p align="center">
    <em>Figure 12: edited ssh config file.</em>
</p>

For the last step, the port 4242 must be open over tcp by the firewalld program. Use the next command to add permanently (on disk) the 4242 port:

<code>firewall-cmd --permanent --add-port=4242/tcp</code>

<strong>NOTE</strong>: the command need super user level access for it to be used.

In case of the necessity to remove a certain port, just substitute the --add-port in the command above to --remove-port.

Check if the configuration was successful with the first command below, shown in Figure 13, and check all the services and ports available with the second command:

<code>firewall-cmd --list-port</code>

<code>firewall-cmd --list-all</code>

<p align="center">
  <img src="https://github.com/RafaelyRezende/Born-4-2beroot/blob/main/rocky_guide/rocky_install18.png" width=50% height=50% alt="ssh config file.">
</p>
<p align="center">
    <em>Figure 13: firewall config check.</em>
</p>

Verify the status of the service with the systemctl utility command. The SSH service probably will be already running, but in case there is a problem or it is disable, use:

<code>systemctl status sshd</code>

<code>systemctl start sshd</code>

<code>systemctl enable sshd</code>

<strong>NOTE</strong>: this command will be useful throughout the journey in system administration, it is possible to list more than one service at a time, just list the services to be checked one after the other separated by spaces like so:

<code>systemctl status service_name_1 service_name_2 service_name_3</code>

### Hostname

At server installation the default name for the machine is localhost, in order to modify this name use the following command:

<code>hostnamectl set-hostname newhostname</code>

<strong>NOTE</strong>: A reboot is necessary to see if the changes are permanent. Also, changing the hostname can lead to problems with services that utilize the hostname as a parameter in configuration files.

### Users and Groups

This is one of the core requirements for being a system administrator. The operations of create, remove and edit users and groups are essential in managing a server. Start by adding a new user to the server, this can be acomplished by running:

<code>useradd -u 4242 -d /home/username -m username</code>

Some useful flags are the -u flag to set an specific user ID number, the -d flag sets the path of the home directory for the new user (in case the default directory is not desirable) and -m creates the home directory. <strong>NOTE</strong>: if you want to set up default files/directories inside the user's home directory when creating a new user, add the necessary files in the <i>/etc/skel</i> directory.

Together with the username of an user, a password needs to be set in place for the user to access the server. This can be achieved with the command:

<code>passwd username</code>

In case the administrator need to delete an user from the server, run the command:

<code>userdel username</code>

By default, this command does not remove the user's home directory, if it is necessary to delete the user's information, together with its directories and files use the -r flag.

Create groups with the following command:

<code>groupadd groupname</code>

View all listed groups and the users inside any specific group in the server by inspecting the <i>/etc/group</i> directory.

<code>cat /etc/group</code>

Add a user to a spcific group with the usermod command utility. This command can do a variety of tasks related to groups and users, if a username have to be changed it is done via usermod. Explore the functionalities of usermod in the <a href="https://www.man7.org/linux/man-pages/man8/usermod.8.html">manual</a> or <a href="https://www.itzgeek.com/how-tos/linux/how-to-modify-user-accounts-in-linux-using-usermod-command.html">here</a>.

<code>usermod -a -G groupname username</code>

The -a flag stands for append and the -G flag tells the usermod command to edit groups.

### Secure Password Policy

The server must have a strict password policy in place. The passwords in the server must have a maximum number of days in use, a minimum amount of days between password changes and a number of days warning before a password expires. This specifications can be edited in the <i>/etc/login.defs</i>, shown in Figure 14 below:

<p align="center">
  <img src="https://github.com/RafaelyRezende/Born-4-2beroot/blob/main/rocky_guide/rocky_install21.png" width=50% height=50% alt="ssh config file.">
</p>
<p align="center">
    <em>Figure 14: login.defs final edit.</em>
</p>

Configure the password minimum characters length and other rules in the <i>/etc/security/pwquality.conf</i> file, follow the instruction in it.

Navigate to the <i>/etc/pam.d/system-auth</i> and <i>/etc/pam.d/password-auth</i> to update the password settings on <strong>both files</strong>. In the following line, if there are more configurations on the file just add to them the following parameters instructions:

<code>password    requisite    pam_pwquality.so try_first_pass local_users_only retry=3 authtok_type= minlen=10 ucredit=-1 lcredit=-1 dcredit=-1 difok=3 reject_username enforce_for_root</code>

Modify the following line to enforce history checks.

<code>password    sufficient    pam_unix.so remember=7</code>

Alternatively, since these files use the pam_pwquality.so file to load the requisites, by editing the file <i>/etc/security/pwquality.conf</i> the same goal can achieved. For this method, read through the commentaries and check the final configuration file will look something like the Figure 15.

<p align="center">
  <img src="https://github.com/RafaelyRezende/Born-4-2beroot/blob/main/rocky_guide/rocky_install20.png" width=50% height=50% alt="ssh config file.">
</p>
<p align="center">
    <em>Figure 15: pwquality file config.</em>
</p>

### Sudo Configuration

The policy for the sudo command is editted in the <i>/etc/sudoers</i> file, however modifying this file raw with the text editor of your choice can make the system brake if wrong modifications are done and there are syntax errors, for example. So, use the command:

<code>visudo</code>

This 'visudo' command opens the <i>/etc/sudoers</i> file where it can edit for specific permissions and configuration safely. Now, add or edit the following lines:

<code>Defaults      passwd_tries=3</code>

<code>Defaults      log_input</code>

<code>Defaults      log_output</code>

<code>Defaults      iolog_dir=/var/log/sudo/</code>

<code>Defaults      logfile=/var/log/sudo/sudo.log</code>

<code>Defaults      requiretty</code>

<code>Defaults      secure_path="/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin:/snap/bin"</code>

The expected file configuration must be similar to the Figure 16, shown bellow:

<p align="center">
  <img src="https://github.com/RafaelyRezende/Born-4-2beroot/blob/main/rocky_guide/rocky_install22.png" width=50% height=50% alt="ssh config file.">
</p>
<p align="center">
    <em>Figure 16: visudo config file.</em>
</p>

<strong>NOTE</strong>: Reset all the passwords to comply with the new policy and check if the rules are being enforced.

### Monitor & Verify

The main goal of this section is to introce the reader to the bash scripting language although it will not in depth about the particularities of the language or either specifically explain each command used in the script for the contents of these subjects can spawn guides of their own. I suggest using the 'man' command line utility to search specific terminal commands used to fetch the necessary information and process it.

Beyond the script to monitor system specifications and resources usage, there is the need to schedule such task so it can be automated. As the system administrator of the server, there is the need of "programatically schedule tasks to be executed at specific intervals" (<a href="https://www.redhat.com/en/blog/linux-cron-command">source</a>). This tasks can be regular updates, backups, system logs management or simple monitoring tasks.

The script in bash should look something like Figure 17. The important commands that will be used to fetch the necessary information are: 

| Command    | Description                                                                                                                                                                                                          |
|------------|----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| uname      | Print some system information.                                                                                                                                                                                       |
| lscpu      | Display information about the CPU architecture.                                                                                                                                                                      |
| free       | Display the total amount of free and used physical and swap memory in the system.                                                                                                                                    |
| df         | Display the amount of disk space available on the file system containing each file argument.                                                                                                                         |
| top        | It provides a dynamic real-time view of a running system. It can display system summary information.                                                                                                                 |
| uptime     | Give a one line display of the following information. The current time, how long the system has been running, how many users are currently logged on, and the system load averages for the past 1, 5 and 15 minutes. |
| ip         | show/manipulate routing, network devices, interfaces and tunnels.                                                                                                                                                    |
| ss         | Another utility to investigate sockets, used to dump socket statistics.                                                                                                                                              |
| lsblk      | List block devices.                                                                                                                                                                                                  |
| journalctl | Query the systemd journal.    

<p align="center">
  <img src="https://github.com/RafaelyRezende/Born-4-2beroot/blob/main/rocky_guide/rocky_install23.png" width=50% height=50% alt="ssh config file.">
</p>
<p align="center">
    <em>Figure 17: monitoring script example template.</em>
</p>

Check if the cronie package is installed. If not installed, run the following command:

<code>dnf install cronie</code>

Start the service and enable it with the 'systemctl' tool in the same way as the [SSH section](#ssh-setup).

<code>systemctl start crond</code>

<code>systemctl enable crond</code>

Once the service start, edit the cron cofiguration file and add the script to be run. Access the cron file through the following command:

<code>crontab -e</code>

The syntax of cron is simple yet it can cause some confusion the first time you see it. A cron job can be scheduled by utilizing the following syntax: "* * * * * bash /path/to/some/script.sh". Each asterisk represent one field of time, minutes, hours, day of the month, month, day of the week, respectively. The asterisk means expand to all values for the field, the comma can be used as a list separator, the '-' as a range separator and '/' as a step for ranges. For example, schedule a job to run the 'annoying_script.sh', located at <i>/usr/local/bin/</i>, every 5 minutes of every monday of April: 

<code>*/5 * * 4 1 bash /usr/local/bin/annoying_script.sh</code>

## Mandatory Check

Run the commands following commands and make sure it is consonant with Figure 18.

<strong>NOTE</strong>: If there is another service running on port 323 of the server, disable it the same way any other service would be stopped and disabled.

<p align="center">
  <img src="https://github.com/RafaelyRezende/Born-4-2beroot/blob/main/rocky_guide/rocky_install24.png" width=50% height=50% alt="ssh config file.">
</p>
<p align="center">
    <em>Figure 18: verification commands and outputs.</em>
</p>

## Bonus Services

### Lighttpd

Lighttpd is an open-source, secure, lightweight and fast web server optmized for high-performance environments with a low memory footprint and minimum CPU load. This software is suitable for static web pages and server that have issues with load problems. It support a wide range of features such as <a href="https://en.wikipedia.org/wiki/Load_balancing_(computing)">load balacing</a>, <a href="https://en.wikipedia.org/wiki/FastCGI">FastCGI</a>, <a href="https://en.wikipedia.org/wiki/Proxy_server#Web_proxy_servers">HTTP proxy</a>, <a href="https://en.wikipedia.org/wiki/WebSocket">WebScokets</a> and <a href="https://en.wikipedia.org/wiki/Lighttpd">more</a>.The documentation of the lighttpd server can be found <a href="https://redmine.lighttpd.net/projects/lighttpd/wiki">here</a> if any problem arises.

### MariaDB

Another open source software, focused on the free availability of a relational databse that can perform with standard proprietary databases. Also, it is built for permance and stability, with integrated cloud services and compatibility features with <a href="https://www.oracle.com/database/">Oracle Database</a> and <a href="https://learn.microsoft.com/en-us/sql/relational-databases/tables/temporal-tables?view=sql-server-ver16">Temporal Data Tables</a>. It is a fork of the <a href="https://www.mysql.com/">MySQL</a> relational database, created bu the original developers of MySQL, after its acquisition by oracle <a href="https://en.wikipedia.org/wiki/Oracle_Corporation">Oracle Corporation</a>.

### PHP

<a href="https://www.php.net/">Personal Home Page</a> (Hypertext Preprocessor) is primarily a server-side scripting language. As a C-based programming language, it is high performance, integrates with the most common web servers and has an extensive library of built-in functions and extensions that facilitates web development. The PHP programming language is the backbone of <a href="https://en.wikipedia.org/wiki/Content_management_system">Content Management Systems</a> (CMS) such as <a href="https://wordpress.org/php">WordPress</a>, Joomla, Drupal. It also has a high performance when dealing with dynamic websites that heavily rely on database connection, user interaction and dynamic content display.

The setup of the PHP and the fastCGI module is requisite for the integration with MariaDB and WordPress.

### WordPress

A Content Management System where the user can edit, administer and manage a webpage in a graphical interface.

## Full Stack Setup

Install the <a href="https://linuxreviews.org/The_Extra_Packages_for_Enterprise_Linux_(EPEL)_repository">Extra Package for Enterprise Linux</a> (EPEL) which contains the tools to connect to other open-source repositories. Use the command bellow and afterwards install the necessary packages for all the above server services.

<code>sudo dnf install epel-release</code>

<code>dnf install -y lighttpd lighttpd-fastcgi mariadb mariadb-server php php-mysqlnd php-fpm php-gd php-xml php-mbstring wget unzip</code>

In sequence, set up the lighttpd service by starting and enabling the service with the systemctl command tool.

<code>systemctl start lighttpd</code>

<code>systemctl enable lighttpd</code>

Next, edit the configuration file for the lighttpd service, path of said file is <i>/etc/lighttpd/lighttpd.conf</i>. The configuration file must be ismilar to Figure 19. Check the 'server.bind' IP address (it must be the IP of the VM) and add the this line to include the 'fastcgi.conf' file for the next steps (this line probaly will be commented out, it can be just uncommented).

<code>include "conf.d/fastcgi.conf"</code>

Edit the modules files of lighttpd, <i>/etc/lighttpd/modules.conf</i>, add "mod_fastcgi" under "mod_access".

<p align="center">
  <img src="https://github.com/RafaelyRezende/Born-4-2beroot/blob/main/rocky_guide/rocky_install25.png" width=50% height=50% alt="ssh config file.">
</p>
<p align="center">
    <em>Figure 19: lighttpd configuration.</em>
</p>

Open the port, listed in the configuration file above, in the firewall and reload it.

<code>firewall-cmd --permanent --add-service=http</code>

<code>firewall-cmd --reload</code>

Manage the SELinux context types:

<code>semanage permissive -a httpd_t</code>

Edit <i>/etc/lighttpd/conf.d/fastcgi.conf</i> and put the following at the bottom of the file:

<br>

    fastcgi.server += ( ".php" =>((
          "host" => "127.0.0.1",
          "port" => "9000",
          "broken-scriptfilename" => "enable"
      )))

<br>

In sequence, set up the lighttpd service by starting and enabling the service with the systemctl command tool.

<code>systemctl start lighttpd</code>

<code>systemctl enable lighttpd</code>

Initialize the MariaDB service with the following command:

<code>systemctl enable --now mariadb</code>

Use the next command to safely start your installation. This command will prompt a series of options, follow the instructions and enter in the database.

<code>mysql_secure_installation</code>

Enter the MariaDB program and create a new database. Afterwards, create a new user and give it all the privileges.

<br>

>  CREATE DATABASE database42;
>
>  CREATE USER 'user'@'localhost' IDENTIFIED BY 'password';
>
>  GRANT ALL PRIVILEGES ON database42.* TO 'user'@'localhost';
> 
>  FLUSH PRIVILEGES;
> 
>  EXIT;

<br>

With MariaDB and Lighttpd configured and running, the service to connect it all and finish with the WordPress website is the configuration of the 'PHP-FPM', Figure 20. Navigate to the <i>/etc/php-fpm.d/www.conf</i> file and edit the 'user', 'group' fields to be equal to 'lighttpd', the 'listen' field to be '127.0.0.1:9000', add 'lighttpd' to 'listen.acl_users' (if there are more services listed in this field like apache and nginx, remove them), and 'listen.allow_clients' to be equal to '127.0.0.1'.

If you are using VIM (as you should), the configuration to be edited are in lines 24, 26, 38, 55, 64, respectvely.

<p align="center">
  <img src="https://github.com/RafaelyRezende/Born-4-2beroot/blob/main/rocky_guide/rocky_install26.png" width=50% height=50% alt="ssh config file.">
</p>
<p align="center">
    <em>Figure 20: php-fpm configuration file part 1.</em>
</p>

<p align="center">
  <img src="https://github.com/RafaelyRezende/Born-4-2beroot/blob/main/rocky_guide/rocky_install27.png" width=50% height=50% alt="ssh config file.">
</p>
<p align="center">
    <em>Figure 21: php-fpm configuration file part 2.</em>
</p>

Finally, for the set up of a WordPress website. Download from the WP website the latest tar with the configuration files needed to the /tmp directory. Next, create a directory on the <i>/var/www/lighttpd/</i> directory to be used to hold the WordPress files. Adjust the permission and the ownership of the directory and files inside it.

>  cd /tmp/
> 
>  wget https://wordpress.org/latest.tar.gz
> 
>  tar -xzvf latest.tar.gz
> 
>  mkdir /var/www/lighttpd/wp_dir
> 
>  mv wordpress/* /var/www/lighttpd/wp_dir
> 
>  chown -R lighttpd:lighttpd /var/www/lighttpd
> 
>  chmod -R 775 /var/www/lighttpd

Now it should be possible to access in the browser the website configuration page of WordPress by entering the IP address of the server with the directory created, for example http://10.11.242.240/wp_dir. Follow the steps propmt in the webpage and input the MariaDB database created in the previous steps.

<strong>NOTE</strong>: if you have any issue, go to the documentation of the service you are having problems, another way to check is seeing the logs of the console or by disabling SELinux enforcing temporarily.

### Extra Service

All the required services are now installed and running, the only thing left to be incorporated into the server is the additional service of your choice! I installed and set up an e-mail service, however you can choose and pick any service that would be useful in a server. This could be a monitoring service such as <a href="https://idroot.us/install-prometheus-rocky-linux-9/">Prometheus</a> or <a href="https://www.linuxboost.com/how-to-install-logwatch-on-rocky-linux/">Logwatch</a>, a service to improve even further the security of the server like Fail2ban, a backup solution for efficient backup, encryption and automation, as <a href="https://linuxconfig.org/how-to-create-secure-and-efficient-backups-with-restic">Restic</a>/<a href="https://installati.one/install-borgbackup-rockylinux-8/">BorgBackup</a>, or another communication protocol such as <a href="https://www.linuxteck.com/how-to-set-up-ftp-server-in-rocky-linux/">FTP</a>, <a href="https://bio-famous.com/how-to-install-and-configure-ipfs-on-linux-system/">IPFS</a>, ...

## Wrap up

By now, if you have reached this far into the document, you can consider yourself brave and full of endurence. I hope you have learned useful knowledge for your programming walk and if I wrote outright wrong information or innacurate takes, please issue a ticket for me to correct it!

<strong>Thank you for taking your time to read this document/guide.</strong>

If you have any doubts or suggestions for improving the guide, please, reach me in a email.
