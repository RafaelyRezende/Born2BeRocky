# Born 2 Be Root

<br>

## Index of Contents

## Prelude

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

NOTE: the swap volume has a unique filesystem type.

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

Generate a SSH key pair for the VM. This is the standard process, use the ssh-keygen command and follow the instructions prompted by the program.

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

NOTE: the command need super user level access for it to be used.

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

NOTE: this command will be useful throughout the journey in system administration, it is possible to list more than one service at a time, just list the services to be checked one after the other separated by spaces like so:

<code>systemctl status service_name_1 service_name_2 service_name_3</code>

### Hostname

At server installation the default name for the machine is localhost, in order to modify this name use the following command:

<code>hostnamectl set-hostname newhostname</code>

NOTE: A reboot is necessary to see if the changes are permanent. Also, changing the hostname can lead to problems with services that utilize the hostname as a parameter in configuration files.

### Users and Groups

This is one of the core requirements for being a system administrator. The operations of create, remove and edit users and groups are essential in managing a server. Start by adding a new user to the server, this can be acomplished by running:

<code>useradd -u 4242 -d /home/username -m username</code>

Some useful flags are the -u flag to set an specific user ID number, the -d flag sets the path of the home directory for the new user (in case the default directory is not desirable) and -m creates the home directory. NOTE: if you want to set up default files/directories inside the user's home directory when creating a new user, add the necessary files in the <i>/etc/skel</i> directory.

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

The server must have a strict password policy in place. The passwords in the server must have a maximum number of days in use, a minimum amount of days between password changes and a number of days warning before a password expires. This specifications can be edited in the <i>/etc/login.defs</i>.

Configure the password minimum characters length and other rules in the <i>/etc/security/pwquality.conf</i> file, follow the instruction in it.

Navigate to the <i>/etc/pam.d/system-auth</i> and <i>/etc/pam.d/password-auth</i> to update the password settings on <strong>both files</strong>. In the following line, add the following instructions:

<code>password    requisite    pam_pwquality.so try_first_pass local_users_only retry=3 authtok_type= minlen=10 ucredit=-1 lcredit=-1 dcredit=-1 difok=3 reject_username enforce_for_root</code>

Modify the following line to enforce history checks.

<code>password    sufficient    pam_unix.so remember=7</code>

NOTE: Reset all the passwords to comply with the new policy and check if the rules are being enforced.

### Sudo Configuration

The policy for the sudo command is editted in the <i></i>

<code>visudo</code> command opens the /etc/sudoers file in our system where you can edit for specific permissions and configuration.
