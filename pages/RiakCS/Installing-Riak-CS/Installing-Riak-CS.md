# Installing Riak CS
A fully functional Riak CS system is comprised of Riak CS, Stanchion, and Riak. Riak CS runs only on 64‐bit platforms. The supported operating systems include Ubuntu 10.04, Ubuntu 11.04, CentOS 5, and CentOS 6. Riak CS is not supported on Microsoft Windows. You can install Riak CS on a single node or using an automated deployment tool.

For those of you like videos, here's a [[video|http://player.vimeo.com/video/42654313]] that demonstrates a typical Riak CS installation.

<div class="note"><div class="title">Note</div>Replace the example filenames in the commands with the filename for the version you want to install. For example, replace w.x.y‐z in the commands below with the Riak CS version number you want to install, such as riak‐cs_0.1.0‐1_amd64.deb.</div>

## Installing Riak CS on a Node
Package files are available for download through the Zendesk system. If you need assistance with downloading files, please [[contact Basho technical support|http://help.basho.com]].  To install a pre‐built Riak CS package, use the commands in the section for your operating system

### Installing Riak CS on Ubuntu
The following command installs Riak CS on a machine running either Debian or Ubuntu.

```bash
sudo dpkg -i riak-cs_w.x.y-z_amd64.deb
```

### Installing Riak CS on CentOS
The following command installs Riak CS on a machine running CentOS.

```bash
sudo rpm -Uvh riak-cs_w.x.y-z_el5.rpm
```

CentOS enables SE Linux by default. If you encounter errors during installation, try disabling SE Linux.

## Installing Stanchion
In a Riak CS system, Stanchion is installed on only one of the nodes in the system. Running Stanchion on more than one node can lead to problems if Riak CS nodes are configured to communicate using multiple Stanchion nodes. In this situation, the uniqueness of bucket names and user email addresses might not be enforced, which, in turn, could lead to unexpected behavior. Use the commands in the section for your operating system to install a pre‐built Stanchion package on the node you choose for Stanchion. 

<div class="note"><div class="title">Note</div>Replace the example filenames in the commands with the filename for the version you want to install. For example, replace w.x.y‐z in the commands below with the Stanchion version number you want to install, such as stanchion_0.1.0‐1_amd64.deb</div>

### Installing Stanchion on Ubuntu
The following command installs Stanchion on a machine running Ubuntu.

```bash
sudo dpkg -i stanchion_w.x.y-z_amd64.deb
```

### Installing Riak CS on CentOS

The following command installs Stanchion on a machine running either Red Hat linux or CentOS.

```bash
sudo rpm -Uvh stanchion_w.x.y-z_el5.rpm
```

<div class="note"><div class="title">Note</div>CentOS enables SE Linux by default. If you encounter errors during installation, try disabling SE Linux.</div>

##Installing Riak
If you have not yet installed Riak, follow [[the instructions|http://wiki.basho.com/Installation.html]] to do so.
