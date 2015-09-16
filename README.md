My personal sysadmin/devops notes and code snippets.

Other usefull files
===================

* [Reading List](reading_list.md)  
* [Performance Tests](performance_tests.md)

Table of content
================

1. [Openstack](#openstack)
2. [CentOS](#centos)
2. [Linux](#linux)  
3. [Spark](#spark)  
4. [bash](#bash)  
5. [vim](#vim)  
6. [Python](#python)  
7. [Ansible](#ansible)  
8. [Git](#git)  
9. [Tmux](#tmux)  
10. [Chef](#chef)  
11. [Sublime Text 3](#sublime-text-3)  
12. [Minotaur](#minotaur)  
13. [Etcd](#etcd)  
14. [Vagrant](#vagrant)  
15. [Docker](#docker)  
16. [Mesos](#mesos)  
17. [Utils](#utils)  
18. [AWS](#aws)
19. [Temporary](#temp)


Openstack
=========

##### Nova cli referance
```sh
nova image-list
nova flavor-list
nova keypair-list
nova secgroup-list-rules
nova list
nova boot
nova volume-list = cinder list
```

##### Look to some logs for find what hypervisor is used in openstack
```sh
cat /var/log/syslog | grep -i kvm
lsusb
lspci
dmesg | grep kvm
cat /proc/cpuinfo /proc/meminfo
```

##### Get access to configaration drive on Open Stack
```sh
mkdir -p /mnt/config
mount /dev/disk/by-label/config-2 /mnt/config
```

##### nova boot <options>
```sh
--config-drive=true     to enable confdrive
--key-name <key_name>       add key pair
--user-data <usr_data_file> add usrdata file which will be executed on instance build in cloud init final module
--flavor <flavor_if>        choose hardware options
--file <dst_file>=<src_file>    transmit file to instance
--meta <meta_option>        add some meta options
--image <image_name_or_id>  chose image which will be booted
```

> Error 500 - check name of security group for spelling!

##### OpenStack components and operands:
```sh
* Keyston - Identity
* Glance - Images
* Nova - Computing
* Neutron - Networking
* Cinder - Volumes
* Swift - Objects(Files)
* Heat - Orchestration(Templates)
* Ceilometer - Telemetry(Monitoring)
* Trove - Databases
* Horizon - Dashboard
```

##### List of all ovs bridges
```sh
ovs-vsctl list-br
```

##### List of ports on ovs bridge <bridge_name>
```sh
ovs-vsctl list-ports <bridge_name>
```

> OVS switch must be on public interface not internal, but if it is set on public interface ssh session will drop, so we need some workaround.  
> We will need another playbook to check up on all networking settings like hostnames, interrelations, etc.  
> Every node needs up to 3 interfaces, public, internal, and one for ceph traphic. Public is needed on network and probably storage nodes.  
> OVS br-ex port must be public - think about creating some solution to this problem!  

Centos
======

##### Add epel repo to Centos 7
sudo yum install epel-release 

##### Fixing SELinux problems for some application e.g. nginx
```sh
sudo yum install policycoreutils-python-2.2.5-11.el7.x86_64
seallow () {
  sudo cat /var/log/audit/audit.log | grep $1 | grep denied | audit2allow -M my${1}
  sudo semodule -i my${1}.pp
}
seallow rabbit
```

#####

```
sudo cat /var/log/audit/audit.log | grep nginx | grep denied | audit2allow -M mynginx2
sudo semodule -i mynginx2.pp
```

##### Bash scripts and function for creating .pp module from .te file
```sh
checkmodule -M -m -o postfixlocal.mod postfixlocal.te
semodule_package -o postfixlocal.pp -m postfixlocal.mod
semodule -i postfixlocal.pp 
```
```
createpp () {
  checkmodule -M -m -o ${1}.mod ${1}.te
  semodule_package -o ${1}.pp -m ${1}.mod
  semodule -i ${1}.pp
}
```

##### SELinux for rabbitmq
```
module rabbit-sensu 1.0;

require {
        type initrc_var_run_t;
        type unreserved_port_t;
        type rabbitmq_beam_t;
        class tcp_socket name_bind;
        class file { read getattr };
}

#============= rabbitmq_beam_t ==============
allow rabbitmq_beam_t initrc_var_run_t:file { read getattr };

#!!!! This avc can be allowed using the boolean 'nis_enabled'
allow rabbitmq_beam_t unreserved_port_t:tcp_socket name_bind;
```

##### SELinux for nginx uchiwa+api proxy
```
module nginx-sensu 1.0;

require {
        type var_run_t;
        type ntop_port_t;
        type tram_port_t;
        type httpd_t;
        type unreserved_port_t;
        class process setrlimit;
        class tcp_socket { name_bind name_connect };
        class file { read write };
}

#============= httpd_t ==============

#!!!! This avc can be allowed using the boolean 'httpd_can_network_connect'
allow httpd_t ntop_port_t:tcp_socket name_connect;
allow httpd_t tram_port_t:tcp_socket name_connect;

#!!!! This avc can be allowed using the boolean 'httpd_setrlimit'
allow httpd_t self:process setrlimit;
allow httpd_t var_run_t:file { read write };

#!!!! This avc can be allowed using the boolean 'nis_enabled'
allow httpd_t unreserved_port_t:tcp_socket name_bind;
```

Linux
=====

##### Connect ot serial port with 9600 baudrate 
1. minicom
  ```sh
  minicom -D /dev/ttyS0 -b 9600
  ```
2. screen
  ```sh
  screen /dev/ttyS0 9600,cs8
  ```
3. python
  ```sh
  screen /dev/ttyS0 9600
  ```

##### `-r` flag in `sed` to use extended regular expressions (`+` for example)
```sh
sed -r
```

##### Append after match
```sh
cat File | sed -r '/[a-zA-Z]*/ a\1'
```

#### Prepend before match
```sh
cat File | sed -r '/[a-zA-Z]*/ i\1'
```

##### Get ip address via wich you can reach internet
```sh
ip route get 8.8.8.8 | awk 'NR==1 {print $NF}'
```

##### get ip addr of interface e.g. wlan0
```sh
ip addr | awk '/inet/ && /wlan0/{sub(/\/.*$/,"",$2); print $2}'
```

##### Add route to default gateway
```sh
route add default gw 10.0.2.1
```

##### edit dns file
```sh
vi /etc/hosts
```

##### Check free disk space
```sh
df -h
```

##### Check disk usage in curretn directory
```sh
du -sh
```

##### Checking cron jobs
```sh
crontab -l
```

##### Top 10 largest files:
```sh
du -a /var | sort -n -r | head -n 10
```

##### Lists all dirs in current dir with their disk space usage
```sh
sudo du -sh $( ls -d */ )
```

##### PostgreSQL command reference(psql):
```sh
\list           lists all databases
\dt         lists all tables in the current database
\c db_name      connects to database db_name
\?          slash commands helper
```

##### list of all active processes
```sh
ps aux
```

##### add key_name to ssh keys
```sh
ssh-add <key_name>
ssh -i <key_name> user@server # ssh-ing using key_name
```

##### Path to vagrant ssh keys
```sh
cd ~/.vagrant.d/insecure_private_key
```

##### How to check open (tcp|udp) ports:
```sh
sudo nmap -sT -O localhost (tcp only)
netstat -tulpn (all ports)
netstat -tln (for tcp only)
```

##### Networking files
* dns config `/etc/resolv.conf`
* config of particular interface `/etc/network/interfaces.d/*.cfg`
* default interfaces config `/etc/network/interfaces`
* dns list `/etc/hosts`

##### Clear swap
```sh
sudo swapoff -a
sudo swapon -a
```

##### Will show time of all reboots
```sh
last reboot
```

##### Will show time of all reboots and user sessions
```sh
last
```

##### Date and time of last boot
```sh
who -b
```

##### Create a custom interface for in userspace interaction(program to program, etc.). It can be lik tap and tun(no ethernet headers overhead)
```sh
sudo ip tuntap add dev mytun mode tun user openwsn
sudo ip link set mytun up
sudo ip addr add 10.0.0.1/24 dev mytun
```

##### List of available nics
```sh
lspci | egrep -i --color 'network|ethernet'
```

##### Detailed info about available nics
```sh
sudo lshw -class network
```

##### Packet processing info of nics
```sh
cat /proc/net/dev
```

##### List all partitions:
```sh
vi /proc/partitions
```

##### Partition detatils:
```sh
fdisk -l <partition_name> # if <partition_name> is not specified, command will look in ti /proc/partitions
```

##### Mark partition as swap
```sh
fdisk -t <partitions_name>
```

##### Service name of dnsmasq dhcp server
```sh
sudo service udhcpd status
```

##### Ways to kill process from autostart (service_name may be find with ps aux | grep <name>)
```sh
update-rc.d <service_name> stop
rm /etc/init.d/<service_name>
service <service_name> stop
```

##### To configure a kernel paramether:
```sh
sysctl -w <kernel_paramether>=<value>
```

> To make change made above permanent edit `/etc/sysctl.conf`

> The kernel ring buffer is a data structure that records messages related to the operation of the kernel. 

##### To check kernel ring buffer(contains logs before syslogd and klogd started):
```sh
dmesg
```

##### To check all messages related to kernel
```sh
vi /var/log/messages
vi /var/log/syslog
```

##### Easy check of external ip using opendns and dig utility
```sh
dig +short myip.opendns.com @resolver1.opendns.com
```

### tcpdump quick reference

##### to capature all traffic on the internal bridge interface.
```sh
tcpdump -n -i br-int
```

##### to capture all traffic on the internal bridge interface and dump it to a file named tcpdump.pcap.
```sh
tcpdump -n -i br-int  -w tcpdump.pcap
```

##### to read in a previously created tcpdump file
```sh
tcpdump -r tcpdump.pcap
```

##### to capture all traffic on any interface
```sh
tcpdump -n -i any
```

### Network namespaces quick reference

##### List namespaces
```sh
ip netns
```

##### Show all interfaces inside the namespace
```sh
ip netns exec qrouter-1fabd5f0-f80b-468d-b733-1b80d0c3e80f ip a
```

##### Show all interfaces inside the namespace
```sh
ip netns exec qrouter-1fabd5f0-f80b-468d-b733-1b80d0c3e80f ip a
```

##### Check routing table inside the router namespace     
```sh
ip netns exec qrouter-1fabd5f0-f80b-468d-b733-1b80d0c3e80f ip r
```

##### IP config inside the router namesapce 
```sh
ip netns exec qrouter-1fabd5f0-f80b-468d-b733-1b80d0c3e80f ifconfig
```

##### IP config inside the dhcp namesace
```sh
ip netns exec qrouter-1fabd5f0-f80b-468d-b733-1b80d0c3e80f ifconfig
```

##### Ping the private IP (of the cirros guest)
```sh
ip netns exec qrouter-1fabd5f0-f80b-468d-b733-1b80d0c3e80f ping -c2 30.0.0.7
ip netns exec qrouter-1fabd5f0-f80b-468d-b733-1b80d0c3e80f ping -c2 192.168.122.14
```

##### ssh into openstack cirros guest
```sh
ip netns exec qdhcp-4a04382f-03bf-49a9-9d4a-35ab9ffc22ad ssh   cirros@30.0.0.7
```

##### to find inode number(key to metadata value) of file:
```sh
ls -i
```

##### Show memory mapping of process with <pid> sorted by amount of memory used
```sh
sudo pmap <pid> | sort -r -n -k2
```

##### Squid post script
```sh
sudo chown proxy:proxy /var/log/squid3
sudo chown proxy:proxy /var/spool/squid3
```

##### to create caching directories
```sh
sudo squid -z
```

##### grep not
```sh
grep -v '<pattern>'
```

##### grep and
```sh
grep -E '<pattern1>.*<pattern2>'
```

##### grep or
```sh
grep -E '<pattern1>|<pattern2>'
grep '<pattern1>\|<pattern2>'
```

##### cut characters <n1> to <n2>
```sh
cut -c <n1>-<n2>
```

##### cut lines <n1> and <n2> with space delimeter
```sh
cut -d' ' -s -f <n1>,<n2>
```

##### Best practice for killing squid processes
```sh
ps aux | grep squid | grep -v grep | cut -c 10-14 | xargs kill
```

##### To allow caching on Squid
```sh
cache_dir ufs /var/spool/squid3 100 16 256 #16 dirs with 256 subdirs each
cache_mem 500 MB
cache allow all
```

##### To enable forwarding on interface
```sh
echo 1 > /proc/sys/net/ipv4/conf/eth0/forwarding
```

##### Messing around with port forwarding for squid using iptables
1. Accept

  ```sh
  iptables -t nat -A PREROUTING -p tcp --dport 8443 -j ACCEPT
  iptables -t nat -A PREROUTING -p tcp --dport 8443 -j REDIRECT --to-port 500
  ```
2. Accept + redirect

  ```sh
  iptables -A PREROUTING -t nat -p tcp --dport 8443 -j REDIRECT --to-port 500
  ```
3. proxy

  ```sh
  gid=`id -g proxy`
  iptables -t nat -A OUTPUT -p tcp --dport 80 -m owner --gid-owner $gid -j ACCEPT
  iptables -t nat -A OUTPUT -p tcp --dport 80 -j DNAT --to-port 3128
  ```

##### Save iptables on ubuntu
```sh
iptables-save > /etc/iptables.rules
```

##### Buld options for squid with https for ubuntu
```sh
'--build=x86_64-linux-gnu' '--prefix=/usr' '--includedir=${prefix}/include' '--mandir=${prefix}/share/man' '--infodir=${prefix}/share/info' '--sysconfdir=/etc' '--localstatedir=/var' '--libexecdir=${prefix}/lib/squid3' '--srcdir=.' '--disable-maintainer-mode' '--disable-dependency-tracking' '--disable-silent-rules' '--datadir=/usr/share/squid3' '--sysconfdir=/etc/squid3' '--mandir=/usr/share/man' '--enable-inline' '--enable-async-io=8' '--enable-storeio=ufs,aufs,diskd,rock' '--enable-removal-policies=lru,heap' '--enable-delay-pools' '--enable-cache-digests' '--enable-underscores' '--enable-icap-client' '--enable-follow-x-forwarded-for' '--enable-auth-basic=DB,fake,getpwnam,LDAP,MSNT,MSNT-multi-domain,NCSA,NIS,PAM,POP3,RADIUS,SASL,SMB' '--enable-auth-digest=file,LDAP' '--enable-auth-negotiate=kerberos,wrapper' '--enable-auth-ntlm=fake,smb_lm' '--enable-external-acl-helpers=file_userip,kerberos_ldap_group,LDAP_group,session,SQL_session,unix_group,wbinfo_group' '--enable-url-rewrite-helpers=fake' '--enable-eui' '--enable-esi' '--enable-icmp' '--enable-zph-qos' '--enable-ecap' '--disable-translation' '--with-swapdir=/var/spool/squid3' '--with-logdir=/var/log/squid3' '--with-pidfile=/var/run/squid3.pid' '--with-filedescriptors=65536' '--with-large-files' '--with-default-user=proxy' '--enable-linux-netfilter' 'build_alias=x86_64-linux-gnu' 'CFLAGS=-g -O2 -fPIE -fstack-protector --param=ssp-buffer-size=4 -Wformat -Werror=format-security -Wall' 'LDFLAGS=-Wl,-Bsymbolic-functions -fPIE -pie -Wl,-z,relro -Wl,-z,now' 'CPPFLAGS=-D_FORTIFY_SOURCE=2' 'CXXFLAGS=-g -O2 -fPIE -fstack-protector --param=ssp-buffer-size=4 -Wformat -Werror=format-security' '--enable-ssl'
```

##### Oneliner in bash for logging most memory consuming processes by <pid>
```sh
while true; do sudo pmap <pid> | sort -r -n -k2 | head -2 | grep -v total >> memory.log; sleep 5; done &
```

##### look for io statistics
```sh
iostat
vmdisk -d
```

##### Debugging of a process with <pid> using strace, redirecting output to <output_file> with max string of 80 chars(default is 32)
```sh
strace -p <pid> -o <output_file> -s 80
```

##### Edit crontab
```sh
contab -e
```

##### Dynamicly show file while it is upating
```sh
tail -f
```

##### Oneliner to log top 3 CPU consuming processes
```sh
(date +"%d-%m %H:%M:%S" && sudo ps aux | sort -r -n -k3 | head -3) >> /var/log/topproc.log
```

##### Checking disk space usage tools 
```sh
du -sh *
ls -lah
```

##### extract the lines between 1234 and 5555" in <someFile>
```sh
sed -n '1234,5555p' <someFile>
```

##### To create tar.gz archive
```sh
tar -cvzf mystuff.tar.gz foo.tex fig1.eps fig2.eps
```

### gpg cheatsheet

##### export pubkey
```sh
gpg --armor --export okushchenko@cogniance.com
```

##### encrypt/decrypt file
```sh
gpg --encrypt <file>
gpg --decrypt <file>
```

##### list keys
```sh
gpg --list-key
```

##### Move+create dir oneliner
```sh
sudo mkdir -p ~/configs/vim/; mv ~/.vimrc $_

```
##### Quick time sync on two machines:
```sh
remote_time=`ssh user@machine1 date` && date -s $remote_time

```
##### Forced time update ntp
```sh
sudo service ntp stop && sudo /usr/sbin/ntpdate 0.ubuntu.pool.ntp.org && sudo service ntp start

```
##### Generate ssh-key
```sh
ssh-keygen -t rsa -C "your_email@example.com"
```

##### Add it to your ssh-agent
```sh
ssh-add ~/.ssh/id_rsa
```

##### Find command syntax
```sh
find / -name -type f <file_name> -perm 744 -size +100M
```

##### If screen is messed up clean to use
```sh
echo -e "\033[0m"
/usr/bin/clear
```

##### be careful with this one
```sh
reset
```

##### Recursive diff between directories
```sh
diff -bur folder1/ folder2/
```

##### System benchmarking utility
```sh
sudo apt-get install sysbench
sudo sysbench --test=cpu --cpu-max-prime=20000 run
```

##### Rsync files from one dir to another(archive mode)
```sh
rsync -a dir1/ dir2
```

##### Rsync in verbose + dry-run mode(no changes will be made)
```sh
rsync -anv dir1/ dir2
```

##### Install deb package
```sh
sudo dpkg -i DEB_PACKAGE
```

##### Downloading with curl
```sh
curl -O http://www.openss7.org/repos/tarballs/strx25-0.9.2.1.tar.bz2
```

##### If ui got stuck on Ubuntu
```sh
compiz --replace
```

##### Find all empty files
```sh
find  /path/to/dest -type f -empty
```

##### Add ssh keys and start ssh daemon
```sh
eval `ssh-agent -s`
ssh-add ~/.ssh/id_rsa
/usr/sbin/sshd
```

##### Output date/time in ISO 8601 format up to seconds without time belt
```sh
date -Is | sed 's/+.*//'
```

bash
====

##### Bash special dollar sign shell variables

* Positional parameters $1,$2,$3… and their corresponding array representation, count and IFS expansion $@, $#, and $*.
* $-    current options set for the shell.
* $$    pid of the current shell (not subshell)
* $_    last argument of last command - most recent parameter (or the abs path of the command to start the current shell immediately after startup)
* $IFS  the (input) field separator
* $?    most recent foreground pipeline exit status
* $!    PID of the most recent background command
* $0    name of the shell or shell script
* $#    number of arguments passed to current script
* $* / $@ list of arguments passed to script as string / delimited list

##### To exit when there is a non-zero status of some command in bash script:
```sh
set -e 
```

##### But there is better way in bash
```sh
trap 'do_something" ERR
trap 'exit' ERR
```

##### If statement with search in string
```sh
if [[ "big string!" == *"string"* ]]; then echo "yep"; fi
```

##### For all of the arguments: <command> !*
```sh
ls foo/ bar/
ls !* # Gives the results of ls foo/ bar/
```

##### For just the last argument: <command> !$
```sh
ls foo/ bar/
ls !$ # Gives the results of ls bar/
```

##### If you want a single argument from a list of arguments from the previous command, you can use <command> !!:<argNumber>
```sh
ls foo/ bar/ baz/
ls !!:2 # Gives the results of ls bar/
ls foo/ bar/ baz/
ls !!:1 # Gives the results of ls foo/
```

vim
===

#### Vim custom syntax in editor enabling(example is for json)
```vim
:source $VIMRUNTIME/syntax/json.vim
```

### vim cheatsheet

##### visual mode(after esc)
```sh
ctrl + v
shift + v
```

##### keyword complition in INSERT mode 
```sh
ctrl + p
```

##### switch between windows in vim
```sh
ctrl + ww
```

##### Easy owrds substitution
```sh
# To move between same words press
/<keyword> # to find word
n
ctrl + n
# Then press 'cw' to change this word
cw
# Then press esc to leave INSERT mode, press 'n' and press '.'(dot) to substitute next word
```

##### To return to failed session in vim
```vim
:recover
```

### Vim navigation through windows:

##### Open new <filename>
```vim
:e <filename>
```

##### List open windows
```vim
:ls
```
##### Go to window #2
```vim
:b 2
```

##### While in insert mode put you in command mode for one key press only
```sh
C-o
```

##### Go to the beginnig of the line while in insert mode
```sh
C-o 0
```

##### Go to the first non-whitespace character of the line
```sh
^
```

##### Som usefull tab configuration
```vim
:set expandtab
:set tabstop=4
:set shiftwidth=4
```

##### Temporary disabling auto indent for paste
```vim
:set paste
# after things are done
:set nopaste
```

Python
======

##### File will be closed after actions - Python
```python
with open("myfile","w") as myFile:
    # do something
```

##### Other way
```python
myFile = open("myfile","w")
try:
    # do something with myFile
finally:
    myFile.close()
```

> SQLAlchemy uses SQLite v3 for databases operations, so when trying to open newly created with python script database use "sqlite3" command not just "sqlite" 

> Oauth credentials can be created here: https://console.developers.google.com/project/737742902183/apiui/credential?authuser=0
> OR go to Google developers console -> APIs % auth -> Credentials -> Create new Client ID -> Client ID for native application (Other)

> If 127.0.0.1:9001 give the following page, then everything is ok.
```html
<td class="status"><span class="statusrunning">running</span></td>
```

##### Supervisord settings so it can be accessed through http
```
[inet_http_server]
port = 127.0.0.1:9001
```

##### Info about supervisor API: http://supervisord.org/api.html

##### Try this out to look at docs of function of module
```python
<module>.<function>.__doc__
```

##### To create and dive into test env for python - in this env you can create deprecated dependecies and install libraries with no harm to your /lib
```sh
virtualenv <test_env>
source <test_env>/bin/activate
```

##### print will not place \n(next line) after each statement if you place `,` after print like this
```python
print foo,
```

##### Python test condition and raise an exeption if it failed
```python
assert <condition>
```

##### Python testing of code
```sh
pip install pep8
pip install flake8
pyflakes test.py
pep8 test.py
```

##### Time test in python
```python
timeit.timeit('exp(8, 16)', setup='from test2 import exp', number=10000)
```

##### list python modules
```sh
pip install yolk
yolk -l
pip freeze
```

##### get some object details
```python
import inspect
inspect.getmembers(<some_class_or_object>)
```

##### use next construction for python logging debug
```python
import logging
logging.basicConfig(level=logging.DEBUG)
```

Ansible
=======

##### Specialize user to avoid errors while sshing
```sh
ansible -u <user>
```

##### use other priv-key
```sh
ansible --private-key <private_key> -m <command>
```

##### Loops in ansible

* `with_fileglob:` matches all files in a single directory like - /playbooks/files/fooapp/*
* `with_items:` traverse over variables
* `with_nested:` traverse over a nested variables like item[0] and item[1] 
* `with_dict:` traverse over a list of hashes(dictionary), u may use some like item.key and item.value
* `with_together:` traverse over few lists like ['a','b'], ['1','2'] as a set of lists '(a,1)', '(b,2)'
* `with_subelements:` traverse over a dict looking for matched key like item.0, item.1
* `with_sequence: start:0 end:10 stride:2 format=test%02x` traverse over a list of generated numbers
* `with_random_choise:` makes a random decision upon a variable of a given list
* `until: <conditional_expression>` until True style loop with given amount of retries and delay between retries
* `with_first_found:` chose the first existing file(variable) in a given list(optional custom paths/file values)
* `with_lines:` loop over the results if some program line by line, it is ALWAYS executed on the controle machine, NOT the local machine
* `with_index_items:` loop over a list but you have list index at item.0 and list value at item.1
* `with_flattened:` loops over a list if list, but those nested lists been merged

### About ansible-playbook command:

##### Use `--tags <tag>` with `--list-tasks` option and `--limit <group>` with `--list-host` option to avoid mistakes

##### Use next pattern to dynamicly group hosts by some paramether in playbook
```yaml
tasks:
  - group_by: key = {{ ansible_distribution }}
```

##### Group created by this pattern can be accessed like this:
```yaml
- hosts: CentOS
```

##### To continue with errors:
```yaml
ignore_errors: yes
```

##### To register return values and other output of command do
```yaml
register: <var_with_command_results>
```

##### Specifing failure behaviour
```yaml
failed_when: "'FAILED' in <var_with_command_results>.stderr"
```

##### Specifing change behaviour
```yaml
changed_when: "<var_with_command_results>.rc != 2"
changed_when: False # this command will never change :(
```

##### Don't invoke shell with `echo >>` but use `lineinfile` module from http://docs.ansible.com/lineinfile_module.html

Git
===

##### git cli reference
```sh
git checkout <branch_name> # changes active branch to branch_name
git checkout -b <branch_name> # creates and changes active branch
git branch <branch_name> # creates new branch
git checkout -- <file_name> # discrads changes in file
git status # status of current git branch
git branch --list # list of git branches
git commit -m "<commit_text>" # make a commit with message
git push -u <init_branch> <dest_branch> # pushes changes to dest_branch in upstream mode
git add <file_name> # adds file_name to current commit
git diff <file1> <file2> # shows diff between file1 and file2
git branch -D <branch_name> # delete branch_name
git pull # updates local repo
```

##### git rollback to old commit(or HEAD~1 for rollback by one commit)
```sh
git stash save
git reset --hard <old-commit-id>
git push -f
```

##### While rabasing, next command will list last 10 commits. Delete commits you dont need any more and save.
```sh
git rebase -i HEAD~10
```

##### List all last commits
```sh
git log
```

##### List commits wich are prepared to push
```sh
git log origin/master..HEAD
git log --branches --not --remotes
```

##### to list last local repo changes
```sh
git reflog
```

##### Git merge process
```sh
git checkout master
git pull origin master
git merge <branch to merge with> -m "<merge message>"
git push origin master
```

##### Check is branch up-to-date
```sh
git fetch -v --dry-run
```

##### Stage All
```sh
git add -A
```

##### Stage new and modified, without deleted
```sh
git add .
```
##### Stage modified and deleted, without new
```sh
git add -u
```

##### Apply local changes to another branch
```sh
git stash save
git checkout <branch_name>
git stash pop
```

##### List stash
```sh
git stash list
```

##### List all remotes
```sh
git remote -v
```

##### Adding new upstream remote repo
```sh
git remote add upstream https://github.com/octocat/Spoon-Knife.git
```

##### Syncing local repo with upstream repo
```sh
git fetch upstream
git checkout master
git merge upstream/master
```

##### Delete local branch
```sh
# Merged
git branch -d <branchName>
# Unmerged
git branch -D <branchName>
```

##### Delete remote branch
```sh
git push origin --delete <branchName>
```

##### Colored double sided diff
```sh
icdiff <file_1> <file_2>
git icdiff <file>
```

##### remove file from remote, but keep a local copy
```sh
git rm --cached <filename>
```

Tmux
====

##### tmux cheat sheet

* C-a s - to switch between sessions
* C-a z - to make current window full screen
* C-a a - go to start of the line
* C-e - go to end of the line
* C-w - delete last word

##### .bashrc config for auto tmux start
```sh
if [[ ! $TERM =~ screen ]]; then
    exec tmux
fi
```

Chef
====

##### Install bundler -> install all gem from Gemfile using bundler -> Run librarian-chef install to install all comunity cookbooks from Cheffile
```sh
cd ~/Downloads/dexter/labs/zookeeper/chef
gem install bundler
bundle install
sudo librarian-chef install
```

##### Regarding SSL certificates verification chef 12.0.1 issue
1. Verify all HTTPS connections (recommended)

  ```sh
  ssl_verify_mode :verify_peer
  ```

2. OR, Verify only connections to chef-server

  ```sh
  verify_api_cert true
  ```

##### Manually install gem using rvm
```sh
sudo /usr/local/rvm/rubies/ruby-2.1.5/bin/gem install ipaddr_extensions
```

# Sublime Text 3

##### Toggle side bar
```
<C-kb> (Ctrl+kb)
```

##### To show context help
```
<CS-p> (Ctrl+Shift+p)
```

Etcd
====

##### etcd cli reference
```sh
etcdctl get skydns/stealthly/us-east-1a/us-east-1/bdoss-dev/bastion
dig @localhost SRV bastion.bdoss-dev.us-east-1.us-east-1a.stealthly

dig any joelgm.me +trace +all

etcdctl ls

curl -XPUT http://127.0.0.1:4001/v2/keys/skydns/local/skydns/carbon -d value='{"Port":2003,"Host": "172.17.0.13"}'
dig +short A carbon.skydns.local
```

Vagrant
=======

##### To run vagrant box without reprovisioning
vagrant up --no-provision

##### To provision multiple vagrant boxes in parallel
vagrant status | awk -F' '  '/virtualbox/ {print $1}' | xargs -n 1 -P 8 vagrant up

Docker
======

##### One liner to stop / remove all of Docker containers
```sh
docker stop $(docker ps -a -q)
docker rm $(docker ps -a -q)
```

##### Fix docker SELinux issue
```sh
chcon -Rt svirt_sandbox_file_t /path/to/volume
```

##### Fix docker access to cutom ports(like dns) issue
```sh
iptables -F
```

Mesos
=====

##### Sample marathon container config: nginx-1.json
```json
{
  "id": "nginx-1",
  "container": {
    "type": "DOCKER",
    "docker": {
      "image": "nginx:1.7.7",
      "network": "HOST"
    }
  },
  "instances": 1,
  "cpus": 0.5,
  "mem": 320,
  "ports": [0,80]
}
```
```sh
curl -X POST -H "Content-Type: application/json" http://127.0.0.1:8080/v2/apps -d@nginx-1.json
```

##### Sample marathon container config with port mapping: nginx-2.json
```sh
{
  "id": "nginx-2",
  "container": {
    "type": "DOCKER",
    "docker": {
      "image": "nginx:1.7.7",
      "network": "BRIDGE",
      "portMappings": [
        { "containerPort": 80, "hostPort": 0, "servicePort": 80, "protocol": "tcp" }
      ]
    }
  },
  "instances": 1,
  "cpus": 0.5,
  "mem": 320
}
```
```sh
curl -X POST -H "Content-Type: application/json" http://127.0.0.1:8080/v2/apps -d@nginx-2.json
```

##### Marathon config for dockerized mesos-dns
```json
{
  "id": "mesos-dns",
  "cmd": "/tmp/mesos-dns -v -config=/tmp/config.json",
  "container": {
    "type": "DOCKER",
    "docker": {
      "image": "ubuntu:14.04",
      "network": "BRIDGE",
      "portMappings": [
        { "containerPort": 53, "hostPort": 0, "servicePort": 53, "protocol": "tcp" }
      ]
    },
    "volumes": [
      {
        "containerPath": "/tmp",
        "hostPath": "/usr/local/mesos-dns",
        "mode": "RO"
      }
    ]
  },
  "instances": 1,
  "cpus": 0.5,
  "mem": 400
}
```
```sh
curl -X POST -H "Content-Type: application/json" http://127.0.0.1:8080/v2/apps -d@/tmp/mesos-dns.json
```

##### Marathon config for raw mesos-dns
```sh
{
"cmd": "sudo /usr/local/mesos-dns/mesos-dns -v -config=/usr/local/mesos-dns/config.json",
"cpus": 0.5,
"mem": 400,
"id": "mesos-dns",
"instances": 1,
"constraints": [["hostname", "CLUSTER", "10.0.0.176"]],
"ports": [53]
}

##### Testing mesos-dns docker container
```sh
sudo docker run -v /usr/local/mesos-dns:/tmp mesosphere/mesos-dns /bin/sh -c '/tmp/mesos-dns -v -config /tmp/config.json'
```

##### Add following attribute to mesos-slave config `/etc/mesos-slave/resources` to enable dns port assignment
```sh
ports(*):[31000-32000, 53-53]
```

##### To run mesos-dns
```sh
sudo /usr/local/mesos-dns/mesos-dns -v -config=/usr/local/mesos-dns/config.json
```

##### Run queries on mesos master API
```sh
curl http://${MASTER_IP}:5050/master/state.json
curl http://${MASTER_IP}:5050/master/state.json | jq [.slaves[].hostname] | jq -r 'join(",")'
curl -sSfLk -m 10 -H 'Accept: text/plain' http://${MASTER_IP}:5050/master/state.json | jq .slaves[].hostname
```

Utils
=====

##### Easy to up http server
```sh
python -m SimpleHTTPServer
```

##### Python serial connection
```sh
pip install pyserial
python -m serial.tools.miniterm -h
```

##### Markdown debugging
```sh
pip install grip
grip ~/Documents/os_comms
```

###### p2pcv missing dependencie library
```sh
sudo apt-get install libpulse-dev 
```

AWS
===

##### Check console output of ec2 instance using boto
```python
from boto import ec2
ec2_connection = ec2.connect_to_region("us-east-1", aws_access_key_id='', aws_secret_access_key='')
rs = ec2_connection.get_all_instances()
x = [i for r in rs for i in r.instances if "i-47788517" in str(i)][0]
output = x.get_console_output()
print output
```

##### Copy file from s3 to local machine
```sh
aws s3 cp --region us-east-1 s3://bdoss-deploy/mesos/mesos-dns/mesos-dns /usr/local/bin/mesos-dns
```

##### Run queries against AWS API
```sh
aws ec2 describe-instances --region us-east-1 --filters "Name=tag:Name,Values=mesos-slave.test.bdoss-dev" | grep Host
```

Minotaur
========

##### Basic deployment procedure
```sh
./sns.py -e dummy -r us-east-1 -t cloudformation-notifications
./sns.py -e dummy -r us-east-1 -t autoscaling-notifications
./vpc.py -e dummy -r us-east-1 -c 10.0.0.0/21
./subnet.py -e dummy -r us-east-1 -z us-east-1a -p private -c 10.0.0.0/23
./subnet.py -e dummy -r us-east-1 -z us-east-1a -p public -c 10.0.2.0/24
./nat.py -e dummy -r us-east-1 -z us-east-1a
./bastion.py -e dummy -r us-east-1 -z us-east-1a
./zookeeper.py -e dummy -d test -r us-east-1 -z us-east-1a
```

dummy key is dexter.pub

Commit to minotaur with caution, dont commit iampolicies template without deleting jenkins group

##### After each hadoop redeploy clean zookeeper
```sh
sudo /opt/apache/zookeeper/zookeeper-3.4.6/bin/zkCli.sh
rmr /chef
ls /
```

There must be 2 and only 2 name nodes for HA  
There must be 3 or more journal nodes for ligitimate quorum

```sh
minotaur lab deploy clouderahadoop journalnode -e bdoss-dev -d test -r us-east-1 -z us-east-1a -n 3
minotaur lab deploy clouderahadoop namenode -e bdoss-dev -d test -r us-east-1 -z us-east-1a -n 2
minotaur lab deploy clouderahadoop resourcemanager -e bdoss-dev -d test -r us-east-1 -z us-east-1a
minotaur lab deploy clouderahadoop datanode -e bdoss-dev -d test -r us-east-1 -z us-east-1a
```

Save local changes using `git stash save` and execute `pull upstream`. Create new branch `git checkout -b feature` and start working.

##### To use dev repo
```sh
git clone https://git@github.com/alexgear/minotaur.git \"$REPO_DIR\"\ncd \"$REPO_DIR\"\ngit checkout feature\ncd
```

##### *TODO:* Add public ip association for zookeeper node

##### *TODO:* There are different subversions for mesos and marathon versions. This must be handled dynamicly.

##### Debug mesos slave start
```sh
service mesos-slave start | awk '{ print $4 }' | xargs strace -p 
```

##### Master chef run
```sh
export mesos_version=0.21.0
export marathon_version=0.7.5
export mesos_dns=false
export marathon=true
export aurora=false
export slave_on_master=false
export chronos=true
export zk_version=3.4.6
export mesos_masters=10.0.2.182
export mesos_masters_eip=54.84.22.122
export hosted_zone_name=bdoss.org
export zk_servers=10.0.1.248
export cassandra_servers=10.0.0.251
export kafka_servers=10.0.1.169
export aurora_url=
export spark=true
export spark_version=1.2.1
export spark_url=
export gauntlet=true
chef-solo -c /deploy/repo/labs/mesos/chef/solo.rb -j /deploy/repo/labs/mesos/chef/solo_master.json
```

##### Slave chef run
```sh
export mesos_version=0.21.0
export zk_version=3.4.6
export zk_servers=10.0.0.208
export cassandra_master=10.0.1.84
export kafka_servers=10.0.0.205
export mesos_masters=10.0.2.95
export mesos_masters_eip=54.173.15.68
export hosted_zone_name=bdoss.net
export mesos_dns=false
export gauntlet=true
export mirrormaker=true
chef-solo -c /deploy/repo/labs/mesos/chef/solo.rb -j /deploy/repo/labs/mesos/chef/solo_slave.json
```

##### Straight-forward gauntlet framework run using shell
```sh
# Find zookeeper, kafka and cassandra nodes that belong to the same deployment and environment
export DEPLOYMENT=dev
export ENVIRONMENT=bdoss-dev
NODES_FILTER="Name=tag:Name,Values=zookeeper.$DEPLOYMENT.$ENVIRONMENT"
QUERY="Reservations[].Instances[].NetworkInterfaces[].PrivateIpAddress"
export ZK_SERVERS=$(aws ec2 describe-instances --region "$REGION" --filters "$NODES_FILTER" --query "$QUERY" | jq --raw-output 'join(",")')
NODES_FILTER="Name=tag:Name,Values=cassandra.$DEPLOYMENT.$ENVIRONMENT"
export CASSANDRA_SERVERS=$(aws ec2 describe-instances --region "$REGION" --filters "$NODES_FILTER" --query "$QUERY" | jq --raw-output 'join(",")')
NODES_FILTER="Name=tag:Name,Values=kafka.$DEPLOYMENT.$ENVIRONMENT"
export KAFKA_SERVERS=$(aws ec2 describe-instances --region "$REGION" --filters "$NODES_FILTER" --query "$QUERY" | jq --raw-output 'join(",")')

export REGION=us-east-1
aws s3 cp --region $REGION s3://bdoss-deploy/kafka/mirrormaker/mirror_maker /usr/local/bin/mirror_maker
chmod +x /usr/local/bin/mirror_maker
echo -e "zookeeper.connect=${ZK_SERVERS}:2181" > /tmp/consumer.config
echo -e "metadata.broker.list=${KAFKA_SERVERS}:9092\n\
timeout=10s" > /tmp/producer.config

aws s3 cp --region $REGION s3://bdoss-deploy/mesos/spark/spark-1.2.0-bin-1.0.4.tgz /tmp/spark-1.2.0-bin-1.0.4.tgz
# aws s3 cp --region $REGION s3://bdoss-deploy/mesos/spark/spark-1.2.0.tgz /tmp/spark-1.2.0.tgz
# mkdir /usr/local/spark
tar -xzf /tmp/spark-1.2.0-bin-1.0.4.tgz -C /opt
mv /opt/spark-1.2.0-bin-1.0.4 /opt/spark
# tar -xzf /tmp/spark-1.2.0.tgz -C /opt
# mv /opt/spark-1.2.0 /opt/spark
echo -e "export MASTER=mesos://zk://${ZK_SERVERS}:2181/mesos\n\
export MESOS_NATIVE_LIBRARY=/usr/local/lib/libmesos.so\n\
export SPARK_EXECUTOR_URI=http://d3kbcqa49mib13.cloudfront.net/spark-1.2.1-bin-hadoop2.4.tgz" > /opt/spark/conf/spark-env.sh
echo -e "spark.master mesos://zk://${ZK_SERVERS}:2181/mesos\n\
spark.executor.uri http://d3kbcqa49mib13.cloudfront.net/spark-1.2.1-bin-hadoop2.4.tgz\n\
spark.mesos.coarse true" > /opt/spark/conf/spark-defaults.conf
echo -e "metadata.broker.list=${KAFKA_SERVERS}:9092" >> /opt/gauntlet/producer.properties

cd /opt/gauntlet
./run.sh 1
```

SPARK
=====
```
##### [Spark mirror url](http://apache.ip-connect.vn.ua/spark/spark-1.2.0/spark-1.2.0.tgz)

##### Run spark shell for debugging purpose
```sh
./bin/spark-shell --master mesos://zk://10.0.2.164:2181/mesos
```

##### Testing spark-shell
```sh
# You need to build spark to run spark shell
/opt/spark/bin/spark-shell

# Once in spark-shell run the following skala code
sc.parallelize(1 to 1000).count()
```

Chef
====

##### Clone repository and checkout to development branch
```sh
git node['mesos']['gauntlet']['install_dir'] do
  repository "http://github.com/alexgear/repo"
  action :sync
end

bash 'checkout to development repository' do
  user 'root'
  cwd '/path/to/develop/branch'
  code "git checkout -b develop origin/develop"
  not_if "git status | grep develop", :cwd => '/path/to/develop/branch'
end
```

Zookeeper
=========

##### zookeeper test connection via ruby
```ruby
#!/usr/bin/env ruby
require zookeeper
z = Zookeeper.new("#{`cat /etc/mesos/zk`}")
zdata = Hash.new{ |h,k| h[k] = Hash.new(&h.default_proc) }
data = z.get(:path => zdata['master'])[:data]
puts data
```

Jenkins
=======

##### Fix require tty error
```
visudo
# add the following lines. ALL=(ALL) NOPASSWD: ALL
# Comment this line "Defaults requiretty"
```

##### Give process ability to survive after build is complete
```sh
if [[ $(ps aux | grep node | grep -v grep) == *node* ]]
  then killall node
fi
/usr/local/sbin/daemonize -E BUILD_ID=dontKillMe -E PORT=8000 -o ./api.log -e ./api.err -c ./ /usr/bin/node express
```

Networking
==========

##### SRV DNS records are ultimately cool, but they are not supported by any of modern browsers (there is a feature request on mozilla bugtracker which date from 1999 so there is a very little of hope). They can be used for loadbalancing and port mapping.
```
_sip._tcp.example.com. 86400 IN SRV 0 5 5060 sipserver.example.com.
```

##### Curl give `no route to host` - check iptables(firewalld) rules and add custom like
```
# Squid proxy firewall rule
firewall-cmd --zone=public --add-port=3128/tcp --permanent
# Node express firewall rule
firewall-cmd --zone=public --add-port=3030/tcp --permanent
firewall-cmd --reload 
```

TEMP
====

##### Full text search using grep
```sh
grep -rl "string" /path
```

##### Add box to vagrant
```
vagrant box add trusty https://cloud-images.ubuntu.com/vagrant/trusty/trusty-server-cloudimg-amd64-juju-vagrant-disk1.box
```

##### Easy to use cpu benchmark
```
openssl speed sha1
```

##### Create new Postgresql user (-E --encrypted -P --pwprompt)
```
createuser -E -P graphite
```

##### Create new Postgresql database (-O --owner) 
```
createdb -O graphite graphite
```

##### Add user to group
```
sudo usermod -a -G groupName userName
sudo adduser userName groupName
```

##### Hex dump of file
```
xxd fileName
```

##### [Nginx ssl configuration generator](https://mozilla.github.io/server-side-tls/ssl-config-generator/)

##### Test responses to fake urls
```
curl -L --resolve fake.url.com:80:$(dig +short url.com | tail -n 1) http://fake.url.com -v
```

##### To keep env variables while doing sudo use `sudo visudo` and add the following
```
Defaults env_keep += "http_proxy SOMEOTHERVARIABLES ANOTHERVARIABLE ETC"
```

##### iotop can be run in batch mode instead of the default interactive mode using the -b option. -o is used to show only processes actually doing I/O, and -qqq is to suppress column names and I/O summary. See man iotop for more options.
```
iotop -boqqq
```

##### Using conditionals (if statement) inside of ansible playbook
```yaml
- name: Write swapfile
  command: |
    {% if swapfile_use_dd %}
    dd if=/dev/zero of={{ swapfile_location }} bs=1M count={{ swapfile_size }} creates={{ swapfile_location }}
    {% else %}
    fallocate -l {{ swapfile_size }} {{ swapfile_location }} creates={{ swapfile_location }}
    {% endif %}
  register: write_swapfile
  when: swapfile_size != false
```


##### [Logstash docs](http://logstash.net/docs/1.4.2/)

### List of nice ansible playbooks
* [logstash](https://github.com/valentinogagliardi/ansible-logstash)
* [sensu jonhadfield's fork](https://github.com/jonhadfield/ansible-playbook-sensu/commits/master)
* [sensu my fork](https://github.com/alexgear/ansible-playbook-sensu/tree/develop)
* [rabbit my fork](https://github.com/alexgear/ansible-playbook-sensu/tree/develop)
* [make it possible to add custom props for sensu client](https://github.com/psviderski/ansible-playbook-sensu/commit/3b6ba59959d9d102c05073c0891bc0a49bb951b1)
* [supervisord](https://github.com/zenoamaro/ansible-supervisord)

##### Loop over a dict in jinja2
```
dict.items()
dict.keys()
dict.values()
```

##### Using journalctl
```
journalctl --dmesg
journalctl --output=json-pretty UNIT=firewalld.service
```

##### Efficient mount options
```
mount -o rw,noexec,nodev,noatime,nodiratime,nobarrier
```

##### Use noop (kinda FIFO) scheduler on SSD disks
```
sudo echo noop > /sys/block/<device-name>/queue/scheduler
```

##### Ultra low latency ping
```
ping 10.1.7.218 -i0.000005 -c50000 -s 16
```

##### merge old with new whisper db
```sh
sudo find * -type f -exec whisper-fill.py {} ../graphite_novalocal-a/{} \;
sudo find * -type f -exec rm {} \;
```

##### Network statistics
```
netstat -s
```

##### Validate certificate SSL
```
openssl ocsp -issuer Symantec_Class_3_EV_SSL_CA_G3.crt -CAfile root_plus_intermediates.pem -nonce -cert cert.pem -url http://ocsp.verisign.com
```

##### Test SSL connection for OSCP and other
```
echo QUIT | openssl s_client -connect example.com:443 -status 2> /dev/null
```

##### To test bandwith and latency via tcp - run
```sh
# On fisrt host
qperf
# On second host
qperf $IP1 tcp_bw tcp_lat
```

##### To fix bad blocks
```sh
sudo badblocks -nsv /dev/sda > bad-blocks
fsck -t ext4 -l /tmp/bad-blocks /dev/sda
# If /dev/sda is a system volume
/sbin/shutdown -r -F now
```

##### Create disk partition using parted
```
parted /dev/sdc mklabel msdos
parted /dev/sdc mkpart primary ext4 1 100%
parted print free
parted rm 1
```

##### Calculate number of Placement Groups in ceph cluster
```sh
for i in 0 1 2 3; do echo  "osd.$i=$(sudo ceph pg dump 2>/dev/null | grep '^3.' | awk '{print $15;}' | grep $i | wc -l) pgs"; done
```

##### Inject config in running ceph cluster
```sh
ceph tell osd.* injectargs '--rbd_cache_max_dirty_age = 1'
```

##### Inspect running config of ceph cluster via admin socket
```sh
ceph --admin-daemon /var/run/ceph/ceph-osd.0.asok config show | grep 'rbd_cache_max_dirty_age ='
```

##### Delete a user
```
azure vm extension set $VMNAME VMAccessForLinux Microsoft.OSTCExtensions '1.*' -i '{"remove_user": "usernametoremove"}'
```

##### Reset the SSH key (Create new user with the SSH key)
```
azure vm extension set $VMNAME VMAccessForLinux Microsoft.OSTCExtensions '1.*' -i '{"username": "currentusername", "ssh_key": "contentofsshkey"}'
```

##### Reset the SSH configuration
```
azure vm extension set $VMNAME VMAccessForLinux Microsoft.OSTCExtensions '1.*' -i '{"reset_ssh": "True"}'
```

##### Change ceph journal to tmpfs
```
OSD_ID=1
service ceph stop osd
sed -i 's/^#*\s*#*journal dio = false/        journal dio = false/' /etc/ceph/ceph.conf
ceph-osd --flush-journal -i ${OSD_ID}
rm -r /var/lib/ceph/ceph-${OSD_ID}
mount -t tmpfs -o size=1G,nr_inodes=1k,mode=0700 tmpfs /var/lib/ceph/journal/
mkdir /var/lib/ceph/journal/ceph-${OSD_ID}
ceph-osd --mkjournal -i ${OSD_ID}
service ceph start osd
```

##### HTTP Apache benchmark
```
ab -n 1000 -c 100 http://testserver/
```

##### Mount volumes in openstack using this options to prevent ceph going crazy
```
sudo umount /dev/vdc1
sudo mkfs.xfs /dev/vdc1 -f
sudo mount -o rw,relatime,seclabel,attr2,inode64,noquota /dev/vdc1 /opt/carbon/storage/whisper/
```

##### Port forwarding
```
echo '1' | sudo tee /proc/sys/net/ipv4/conf/eth0/forwarding
iptables -t nat -A PREROUTING -p tcp -i ppp0 --dport 8001 -j DNAT --to-destination 192.168.1.200:8080
iptables -A FORWARD -p tcp -d 192.168.1.200 --dport 8080 -m state --state NEW,ESTABLISHED,RELATED -j ACCEPT
```

##### Utility to performance test graphite
```
~/repos/velux-deploy/graphite_perf_test_go -host=10.3.2.8:5000 -proto=tcp -threads=4 -simul 240 -connections=240 -points 1000
~/repos/velux-deploy/graphite_perf_test_go -host=10.3.2.8:5000 -proto=tcp -threads=4 -simul 1 -connections=1 -points 100000
java -jar ~/repos/graphite-stresser/dist/stresser.jar 10.3.2.8 2203 12 128 1 false
```

##### Easy to use disk stats
```
iostat -c 1 2 /dev/sda1
ioping -c 10 .
```

##### Tweaking kernel and mount options to maximize xfs performance
```sh
sudo mkfs.xfs -f -n size=64k -l size=64m -d agcount=45 /dev/vdb
mount -o remount,nobarrier,logbufs=8,logbsize=256k,allocsize=2M,noatime,nodiratime,delaylog
```

```sh
sudo sysctl vm.dirty_background_ratio=0 vm.dirty_background_bytes=209715200 vm.dirty_ratio=40 vm.dirty_bytes=0 vm.dirty_writeback_centisecs=100 vm.dirty_expire_centisecs=200
```

```
[new]
vm.dirty_background_ratio = 10 -> 0
vm.dirty_background_bytes = 0 -> 209715200 # 200 MBytes
vm.dirty_ratio = 20 -> 40
vm.dirty_bytes = 0 -> 0
vm.dirty_writeback_centisecs = 500 -> 100
vm.dirty_expire_centisecs = 3000 -> 200
```

```sh
echo 2 > /sys/block/sdb/queue/rq_affinity; echo cfq > /sys/block/xvdb/queue/scheduler; echo 256 > /sys/block/xvdb/queue/nr_requests; echo 256 > /sys/block/xvdb/queue/read_ahead_kb
```
