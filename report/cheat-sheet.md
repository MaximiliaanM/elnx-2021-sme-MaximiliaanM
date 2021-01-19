# Cheat sheets and checklists

- Student name: Maximiliaan Muylaert
- Github repo: https://github.com/HoGentTIN/elnx-2021-sme-MaximiliaanM

## Basic commands

| Task                | Command |
| :---                | :---    |
| Query IP-adress(es) | `ip a`  |
| SSH to machine | `ssh <USER_NAME>@<IP>`|
| SSH with vagrant| `vagrant ssh <SERVER_NAME>`
|Install role dependencies| `./scripts/role-deps.sh`|
|Run tests            | `sudo /vagrant/test/runbats.sh`|

---

## Ansible rhbase config

| Task              | Command           | Arguments |
| :---              | :---              | :---      |
| Enable firewall services|`rhbase_firewall_allow_services`| https,  http|
| Enforce SELinux| `rhbase_selinux_state`| enforcing|
| Install the EPEL repository| `rhbase_repositories`| epel-release|
| Install packages| `rhbase_install_packages`| git, nano, vim-enhanced,...|
| Create Users| `rhbase_users`| name, comment, groups, password, ssh_key|
| Allow groups to SSH| `rhbase_ssh_allow_groups`| vagrant|
| Enable motd| `rhbase_dynamic_motd`| true, false|

---

## Git workflow

Simple workflow for a personal project without other contributors:

| Task                                         | Command                   |
| :---                                         | :---                      |
| Current project status                       | `git status`              |
| Select files to be committed                 | `git add FILE...`         |
| Commit changes to local repository           | `git commit -m 'MESSAGE'` |
| Push local changes to remote repository      | `git push`                |
| Pull changes from remote repository to local | `git pull`                |

---

## Troubleshooting
### Fase 1: Phisical and Network Access layer

1. Is the power on?
2. Is the network cable plugged in?

    -> Virtualbox -> Settings -> Network -> Advanced: "Cable connected" needs to be checked
    

3. Are Etherport LEDs on?
4. Use command `ip link`

    * Up: ok, interface is connected
    * NO-CARRIER: no signal

### Fase 2: Internet layer

* Network interface should have an IP address
* Default gateway should be set
* DNS server should be set

1. IP address:

    * Check if DHCP has been set automatically or manually:

        -> `cat /etc/sysconfig//network-scripts/ifcfg-IFACE`

    * Use command `ip a` to list IP addresses for each interface

2. Default gateway: `ip r`

    * Default GW present?
    * In correct subnet?
    * Check network configuration 

3. DNS server: `/etc/resolv.conf`

    * nameserver option present?
    * Expected IP?

4. Check LAN connectivity:

    * Ping between hosts
    * Ping default GW & DNS
    * Query DNS (dig, nslookup)


### Fase 3: Transport layer

1. Service running?

    * `sudo systemctl status httpd.service`
   

2. Correct Ports/interfaces?

    * TCP Service: `sudo ss -tlnp`
    * UDP Service: `sudop ss -ulnp`
    * Correct port number?
    
3. Firewall settings:
    * `sudo firewall-cmd --list-all`
    * `sudo firewall-cmd --add-service SERVICE`
    * `sudo firewall-cmd --add-service SERVICE --permanent`
    * `sudo firewall-cmd --reload`

### Fase 4: Application layer

1. General checks
    * Check the logs: `sudo journalctl -f -l -u named`
    * Config file syntax validation:
        * Main config: `sudo named-checkconf /etc/named.conf`
        * Zone files:
            * Forward lookup zone: `sudo named-checkzone example.com /var/named/example.com`
            * Reverse lookup zone: `sudo named-checkzone 2.0.192.in-addr.arpa /var/named/2.0.192.in-addr.arpa`
             

2. Use dig/nslookup:
    * dig @DNS_SERVER_IP HOSTNAME
    * nslookup HOSTNAME DNS_SERVER_IP

### Fase 5: SELinux Troubleshooting

* Settings:
    * Booleans: `getsebool`, `setsebool`
    * Contexts, labels: `ls -z`, `chcon` ,`restorecon`
    * Policy modules: `sepolicy`