# Enterprise Linux Lab Report

- Student name: Maximiliaan Muylaert
- Github repo: <https://github.com/HoGentTIN/elnx-2021-sme-MaximiliaanM>

---

## Test plan

### Start the vm's

To start the vm's execute `vagrant up` in a bash shell.

### Test `pr001` & `pr002`:

SSH in to the vm `pr001` and execute following command:

```bash
sudo /vagrant/test/runbats.sh
```

Do the same for `pr002`

### Output of both vm's

```bash
Running test /vagrant/test/common.bats
 ✓ SELinux should be set to 'Enforcing'
 ✓ Firewall should be enabled and running
 ✓ EPEL repository should be available
 ✓ Bash-completion should have been installed
 ✓ bind-utils should have been installed
 ✓ Git should have been installed
 ✓ Nano should have been installed
 ✓ Tree should have been installed
 ✓ Vim-enhanced should have been installed
 ✓ Wget should have been installed
 ✓ Admin user maximiliaan should exist
 ✓ An SSH key should have been installed for maximiliaan
 - Custom /etc/motd should have been installed (skipped)

13 tests, 0 failures, 1 skipped
Running test /vagrant/test/pr001/masterdns.bats
 ✓ The dig command should be installed
 ✓ The main config file should be syntactically correct
 ✓ The forward zone file should be syntactically correct
 ✓ The reverse zone files should be syntactically correct
 ✓ The service should be running
 ✓ Forward lookups public servers
 ✓ Forward lookups private servers
 ✓ Reverse lookups public servers
 ✓ Reverse lookups private servers
 ✓ Alias lookups public servers
 ✓ Alias lookups private servers
 ✓ NS record lookup
 ✓ Mail server lookup

13 tests, 0 failures
```

---

## Procedure/Documentation

### 1. Adding `pr001` and `pr002` as a vagrant host

To set these as hosts open the `vagrant-hosts.yml` file and enter following code:

```yaml
- name: pr001
  ip: 172.16.128.1
  netmask: 255.255.0.0
- name: pr002
  ip: 172.16.128.2
  netmask: 255.255.0.0
```
This sets the name and ip of our hosts.

### 2. Configure playbook & installing roles

Open the playbook `site.yml` and add the hosts **pr001** and **pr002**. install the roles `bertvv.rh-base` and `bertvv.bind`.

```yaml
- hosts: pr001, pr002
  become: true
  roles:
    - bertvv.rh-base
    - bertvv.bind
```

The roles are now added to our playbook. To install them, open a bash shell in the main directory and run following script: `./scripts/role-deps.sh.`

### 3. Configure host `pr001` and `pr002`

Make 2 new files called `pr001.yml` and `pr002.yml` in `ansible\host_vars`. To configure these files follow next steps (these steps need to be repeated for `pr002`):

1. **Allow dns on the firewall by entering following code:**

    ```yaml
    rhbase_firewall_allow_services:
    - dns
    ```

2. **Configure the hosts that are allowed to query the DNS-server**

    ```yaml
    bind_allow_query:
    - '192.0.2.0/24'
    - '172.16.0.0/16'
    ```

3. **Configure the DNS-server to listen on all interfaces**

    ```yaml
    bind_listen_ipv4:
    - any
    ```

4. **Configure the domains**

    To configure the primary DNS-server use the variable **primary** for the zone parameter `type`: . Use the variable **secondary** to configure as secondary DNS-server.

    ```yaml
    bind_zones:
    - name: avalon.lan
        type: primary #Primary DNS-Server
        create_reverse_zones: true
        primaries: 
        - 172.16.128.1
        name_servers:
        - pr001.avalon.lan.
        - pr002.avalon.lan.
        networks:
        - '203.0.113'
        - '172.16'
    ```

5. **Configure hosts**

    **Note** that `hosts`: is a zone parameter of `bind_zones`:, so you need to enter it in the previous written code.

    ```yaml
    hosts:
        - name: router
            ip: 
            - 203.0.113.254
            - 172.16.255.254
            aliases:
            - gw
        - name: pu001
            ip: 203.0.113.10
            aliases: 
            - www
        - name: pu002
            ip: 203.0.113.20
            aliases: mail
        - name: pr001
            ip: 172.16.128.1
            aliases:
            - ns1
        - name: pr002
            ip: 172.16.128.2
            aliases:
            - ns2
        - name: pr003
            ip: 172.16.128.3
            aliases:
            - dhcp
        - name: pr004
            ip: 172.16.128.4
            aliases:
            - directory
        - name: pr010
            ip: 172.16.128.10
            aliases:
            - inside
        - name: pr011
            ip: 172.16.128.11
            aliases:
            - files
    ```

### 6. Configure MX record pointing to the mail server

**Note** that `mail_servers`: is a zone parameter of `bind_zones`:, so you need to enter it in the previous written code.

```yaml
mail_servers:
  - name: pu002
    preference: 10
```

---

## Test report

### Test results of `pr001`:

```bash
$ vagrant up pr001
Bringing machine 'pr001' up with 'virtualbox' provider...
==> pr001: You assigned a static IP ending in ".1" to this machine.
==> pr001: This is very often used by the router and can cause the
==> pr001: network to not work properly. If the network doesn't work
==> pr001: properly, try changing this IP.
==> pr001: Importing base box 'bento/centos-7.6'...
==> pr001: Matching MAC address for NAT networking...
==> pr001: You assigned a static IP ending in ".1" to this machine.
==> pr001: This is very often used by the router and can cause the
==> pr001: network to not work properly. If the network doesn't work
==> pr001: properly, try changing this IP.
==> pr001: Checking if box 'bento/centos-7.6' version '201907.24.0' is up to date...
==> pr001: Setting the name of the VM: elnx-2021-sme-MaximiliaanM_pr001_1603460245704_62148
==> pr001: Clearing any previously set network interfaces...
==> pr001: Preparing network interfaces based on configuration...
    pr001: Adapter 1: nat
    pr001: Adapter 2: hostonly
==> pr001: Forwarding ports...
    pr001: 22 (guest) => 2222 (host) (adapter 1)
==> pr001: Running 'pre-boot' VM customizations...
==> pr001: Booting VM...
==> pr001: Waiting for machine to boot. This may take a few minutes...
    pr001: SSH address: 127.0.0.1:2222
    pr001: SSH username: vagrant
    pr001: SSH auth method: private key
    pr001: Warning: Connection reset. Retrying...
    pr001: Warning: Connection aborted. Retrying...
    pr001: Warning: Remote connection disconnect. Retrying...
    pr001: Warning: Connection aborted. Retrying...
    pr001: Warning: Connection reset. Retrying...
    pr001: Warning: Remote connection disconnect. Retrying...
    pr001: Warning: Connection aborted. Retrying...
    pr001: Warning: Connection reset. Retrying...
    pr001: Warning: Remote connection disconnect. Retrying...
    pr001:
    pr001: Vagrant insecure key detected. Vagrant will automatically replace
    pr001: this with a newly generated keypair for better security.
    pr001:
    pr001: Inserting generated public key within guest...
==> pr001: Machine booted and ready!
==> pr001: Checking for guest additions in VM...
==> pr001: Setting hostname...
==> pr001: Configuring and enabling network interfaces...
==> pr001: Mounting shared folders...
    pr001: /vagrant => C:/Users/Maximiliaan/Documents/School/3de jaar/EnterpriseLinux/elnx-2021-sme-MaximiliaanM
==> pr001: Running provisioner: ansible_local...
    pr001: Installing Ansible...
    pr001: Running ansible-playbook...

PLAY [pu001] *******************************************************************
skipping: no hosts matched

PLAY [pr001, pr002] ************************************************************

TASK [Gathering Facts] *********************************************************
ok: [pr001]

TASK [bertvv.rh-base : Load distro specific variables] *************************
ok: [pr001] => (item=/vagrant/ansible/roles/bertvv.rh-base/vars/RedHat.yml)

TASK [bertvv.rh-base : include_tasks] ******************************************
included: /vagrant/ansible/roles/bertvv.rh-base/tasks/install.yml for pr001

TASK [bertvv.rh-base : Install | Check minimal value of ‘rhbase_repo_installonly_limit’ (>= 2)] ***
ok: [pr001] => {
    "msg": "The value of ‘rhbase_repo_installonly_limit’ should be at least 2, actual value is 3."
}

TASK [bertvv.rh-base : Install | Ensure the machine-ID is available] ***********
ok: [pr001]

TASK [bertvv.rh-base : Install | Ensure basic systemd services are running] ****
ok: [pr001] => (item=systemd-journald)
ok: [pr001] => (item=systemd-tmpfiles-setup-dev)
ok: [pr001] => (item=systemd-tmpfiles-setup)

TASK [bertvv.rh-base : Install | Role/Ansible dependencies] ********************
ok: [pr001]

TASK [bertvv.rh-base : Install | Package management configuration (yum)] *******
changed: [pr001]

TASK [bertvv.rh-base : Install | Ensure specified external repositories are installed] ***
ok: [pr001]

TASK [bertvv.rh-base : Install | Ensure specified repositories are enabled] ****

TASK [bertvv.rh-base : Install | Ensure specified packages are installed] ******
changed: [pr001]

TASK [bertvv.rh-base : Install | Ensure specified packages are NOT installed] ***
ok: [pr001]

TASK [bertvv.rh-base : Install | Ensure all updates are installed] *************
skipping: [pr001]

TASK [bertvv.rh-base : include_tasks] ******************************************
included: /vagrant/ansible/roles/bertvv.rh-base/tasks/config.yml for pr001

TASK [bertvv.rh-base : Config | Ensure host name is in the hosts file] *********
skipping: [pr001]

TASK [bertvv.rh-base : Config | Install dependency for dynamic MotD] ***********
changed: [pr001]

TASK [bertvv.rh-base : Config | Install script generating dynamic MotD] ********
changed: [pr001]

TASK [bertvv.rh-base : Config | Check if ifup-eth script overrides firewall zone] ***
ok: [pr001]

TASK [bertvv.rh-base : Config | Don’t override firewall zone in ifup-eth script] ***
skipping: [pr001]

TASK [bertvv.rh-base : Config | Check if ifup-post script overrides firewall zone] ***
ok: [pr001]

TASK [bertvv.rh-base : Config | Don’t override firewall zone in ifup-post script] ***
skipping: [pr001]

TASK [bertvv.rh-base : Config | Set the TZ environment variable] ***************
changed: [pr001]

TASK [bertvv.rh-base : Config | Set protocol version for SSH] ******************
changed: [pr001]

TASK [bertvv.rh-base : Config | Set PermitEmptyPasswords] **********************
changed: [pr001]

TASK [bertvv.rh-base : Config | Set IgnoreRhosts] ******************************
changed: [pr001]

TASK [bertvv.rh-base : Config | Set HostbasedAuthentication] *******************
changed: [pr001]

TASK [bertvv.rh-base : Config | Set RhostsRSAAuthentication] *******************
changed: [pr001]

TASK [bertvv.rh-base : include_tasks] ******************************************
included: /vagrant/ansible/roles/bertvv.rh-base/tasks/services.yml for pr001

TASK [bertvv.rh-base : Services | Ensure SSH daemon is running] ****************
ok: [pr001]

TASK [bertvv.rh-base : Services | Ensure `/var/log/journal` exists] ************
changed: [pr001]

TASK [bertvv.rh-base : Services | Ensure specified services are running] *******

TASK [bertvv.rh-base : Services | Ensure specified services are NOT running] ***

TASK [bertvv.rh-base : include_tasks] ******************************************
included: /vagrant/ansible/roles/bertvv.rh-base/tasks/security.yml for pr001

TASK [bertvv.rh-base : Security | Make sure SELinux has the desired state (enforcing)] ***
changed: [pr001]

TASK [bertvv.rh-base : Security | Enable SELinux booleans] *********************

TASK [bertvv.rh-base : Security | Make sure the firewall is running] ***********
changed: [pr001]

TASK [bertvv.rh-base : Security | Make sure basic services can pass through firewall] ***
ok: [pr001] => (item=dhcpv6-client)
ok: [pr001] => (item=ssh)

TASK [bertvv.rh-base : Security | Make sure user specified services can pass through firewall] ***

TASK [bertvv.rh-base : Security | Make sure user specified ports can pass through firewall] ***

TASK [bertvv.rh-base : Security | Make sure specified interfaces are added] ****

TASK [bertvv.rh-base : include_tasks] ******************************************
included: /vagrant/ansible/roles/bertvv.rh-base/tasks/users.yml for pr001

TASK [bertvv.rh-base : Users | Process groups] *********************************
ok: [pr001] => (item={u'comment': u'Administrator', u'password': u'$6$6QhrgSAjVwd2IWk8$pzPgb8nirer5NNCupmzei9jMZp5MjuQfkgLuqahI8SYp1yh93./pEHhT9Dp6rjqqhf.rTdfGyAvgsBtO0PYeP1', u'name': u'maximiliaan', u'groups': [u'wheel'], u'ssh_key': u'ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAACAQDSKv3fs5Mg0SV33eS+ho96xEHByU3SmI0/UMmXmFkmqWJf+TQO44nzNyOwZiAmLXtZnle5QI92X9GfCm77iTEYaye4wLy60NVoyxg+CggOhaIcc6IxBOajV8YoGHVG7/ZwiUXx/b/pzAcJScXe7n+DIxSkyhJ65r6WIoZhjE5phOsKopjRRJwvsfQsSOHapjeOZuSyJD7T4m1PBvrAnGQ6607mF05fG6UPHCdH3zwMnuutzdLllc4FmmvlqRv6FNAziEouZYenHjRMNOoBwW7fK7RiZrECZnaiqyvY3a+PgKXCLcaEcsdYBdjiJFnZtEITuXVJ3kqZdsJNYbn/bhFK29PsywVC9h1+RI5vJy0DGu1FTH5viAYDeHkm4Us7rAwjsplrHEuGLLfpXPlg8FBHPa/LBBVqkcTY/RWyQyWjUUQKq1eSJgzQvfTuKh9RSyszMOERsJ75fLJlT9aSMNdfXeeOfdHZagMGVPhuEmdYEA1rPNUBV7uBsiC+AS8Ip/Ee4+XvnLRALz4cfS8SN/v344uHORKIAtXsdXAI4YIuunBbiA1dY6x2+V4kHBveMgqtnUKwud++O1ADJH/C1CGHEk0cZeshDIIdnUlsTOMieFe3wrATnGP3Rp5sK0j09jn/YxDsHlkeSA9F66oAZxBynyEX9xduYMpRJ+YShMHFeQ== Maximiliaan@MSI'})

TASK [bertvv.rh-base : Users | Add groups] *************************************
ok: [pr001] => (item=wheel)

TASK [bertvv.rh-base : Users | Add users] **************************************
changed: [pr001] => (item={u'comment': u'Administrator', u'password': u'$6$6QhrgSAjVwd2IWk8$pzPgb8nirer5NNCupmzei9jMZp5MjuQfkgLuqahI8SYp1yh93./pEHhT9Dp6rjqqhf.rTdfGyAvgsBtO0PYeP1', u'name': u'maximiliaan', u'groups': [u'wheel'], u'ssh_key': u'ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAACAQDSKv3fs5Mg0SV33eS+ho96xEHByU3SmI0/UMmXmFkmqWJf+TQO44nzNyOwZiAmLXtZnle5QI92X9GfCm77iTEYaye4wLy60NVoyxg+CggOhaIcc6IxBOajV8YoGHVG7/ZwiUXx/b/pzAcJScXe7n+DIxSkyhJ65r6WIoZhjE5phOsKopjRRJwvsfQsSOHapjeOZuSyJD7T4m1PBvrAnGQ6607mF05fG6UPHCdH3zwMnuutzdLllc4FmmvlqRv6FNAziEouZYenHjRMNOoBwW7fK7RiZrECZnaiqyvY3a+PgKXCLcaEcsdYBdjiJFnZtEITuXVJ3kqZdsJNYbn/bhFK29PsywVC9h1+RI5vJy0DGu1FTH5viAYDeHkm4Us7rAwjsplrHEuGLLfpXPlg8FBHPa/LBBVqkcTY/RWyQyWjUUQKq1eSJgzQvfTuKh9RSyszMOERsJ75fLJlT9aSMNdfXeeOfdHZagMGVPhuEmdYEA1rPNUBV7uBsiC+AS8Ip/Ee4+XvnLRALz4cfS8SN/v344uHORKIAtXsdXAI4YIuunBbiA1dY6x2+V4kHBveMgqtnUKwud++O1ADJH/C1CGHEk0cZeshDIIdnUlsTOMieFe3wrATnGP3Rp5sK0j09jn/YxDsHlkeSA9F66oAZxBynyEX9xduYMpRJ+YShMHFeQ== Maximiliaan@MSI'})

TASK [bertvv.rh-base : Users | Set up SSH keys] ********************************
changed: [pr001] => (item={u'comment': u'Administrator', u'password': u'$6$6QhrgSAjVwd2IWk8$pzPgb8nirer5NNCupmzei9jMZp5MjuQfkgLuqahI8SYp1yh93./pEHhT9Dp6rjqqhf.rTdfGyAvgsBtO0PYeP1', u'name': u'maximiliaan', u'groups': [u'wheel'], u'ssh_key': u'ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAACAQDSKv3fs5Mg0SV33eS+ho96xEHByU3SmI0/UMmXmFkmqWJf+TQO44nzNyOwZiAmLXtZnle5QI92X9GfCm77iTEYaye4wLy60NVoyxg+CggOhaIcc6IxBOajV8YoGHVG7/ZwiUXx/b/pzAcJScXe7n+DIxSkyhJ65r6WIoZhjE5phOsKopjRRJwvsfQsSOHapjeOZuSyJD7T4m1PBvrAnGQ6607mF05fG6UPHCdH3zwMnuutzdLllc4FmmvlqRv6FNAziEouZYenHjRMNOoBwW7fK7RiZrECZnaiqyvY3a+PgKXCLcaEcsdYBdjiJFnZtEITuXVJ3kqZdsJNYbn/bhFK29PsywVC9h1+RI5vJy0DGu1FTH5viAYDeHkm4Us7rAwjsplrHEuGLLfpXPlg8FBHPa/LBBVqkcTY/RWyQyWjUUQKq1eSJgzQvfTuKh9RSyszMOERsJ75fLJlT9aSMNdfXeeOfdHZagMGVPhuEmdYEA1rPNUBV7uBsiC+AS8Ip/Ee4+XvnLRALz4cfS8SN/v344uHORKIAtXsdXAI4YIuunBbiA1dY6x2+V4kHBveMgqtnUKwud++O1ADJH/C1CGHEk0cZeshDIIdnUlsTOMieFe3wrATnGP3Rp5sK0j09jn/YxDsHlkeSA9F66oAZxBynyEX9xduYMpRJ+YShMHFeQ== Maximiliaan@MSI'})

TASK [bertvv.rh-base : include_tasks] ******************************************
included: /vagrant/ansible/roles/bertvv.rh-base/tasks/admin.yml for pr001

TASK [bertvv.rh-base : Admin | Make sure users from the wheel group can use sudo] ***
changed: [pr001]

TASK [bertvv.rh-base : Admin | Set attributes of sudo configuration file for wheel group] ***
changed: [pr001]

TASK [bertvv.rh-base : Admin | Make sure only these groups can ssh] ************
changed: [pr001]

TASK [bertvv.rh-base : include_tasks] ******************************************
skipping: [pr001]

TASK [bertvv.bind : Source specific variables] *********************************
ok: [pr001] => (item=/vagrant/ansible/roles/bertvv.bind/vars/RedHat.yml)

TASK [bertvv.bind : Check `primaries` or `forwarders` was set for each zone] ***
ok: [pr001] => (item=avalon.lan)

TASK [bertvv.bind : Update package cache for Debian based distros] *************
skipping: [pr001]

TASK [bertvv.bind : Check that transfer key exists in keys list] ***************
skipping: [pr001]

TASK [bertvv.bind : Install BIND] **********************************************
changed: [pr001] => (item=python-netaddr)
changed: [pr001] => (item=bind)
ok: [pr001] => (item=bind-utils)

TASK [bertvv.bind : Ensure runtime directories referenced in config exist] *****
changed: [pr001] => (item=/var/named/dynamic)
changed: [pr001] => (item=/var/named/data)
changed: [pr001] => (item=/var/named)

TASK [bertvv.bind : Ensure Directory for Cached Secondary Zones exists] ********
changed: [pr001]

TASK [bertvv.bind : Create serial, based on UTC UNIX time] *********************
ok: [pr001]

TASK [bertvv.bind : Create extra config for authenticated XFR request] *********
skipping: [pr001]

TASK [bertvv.bind : Configure zones] *******************************************
included: /vagrant/ansible/roles/bertvv.bind/tasks/zones.yml for pr001

TASK [bertvv.bind : Set list of all host IP addresses] *************************
ok: [pr001]

TASK [bertvv.bind : Read forward zone hashes] **********************************
ok: [pr001] => (item=avalon.lan)

TASK [bertvv.bind : Create dict of forward hashes] *****************************
ok: [pr001] => (item=avalon.lan)

TASK [bertvv.bind : Read reverse ipv4 zone hashes] *****************************
ok: [pr001] => (item=203.0.113)
ok: [pr001] => (item=172.16)

TASK [bertvv.bind : Create dict of reverse hashes] *****************************
ok: [pr001] => (item=avalon.lan)
ok: [pr001] => (item=/var/named/113.0.203.in-addr.arpa)
ok: [pr001] => (item=avalon.lan)
ok: [pr001] => (item=/var/named/16.172.in-addr.arpa)

TASK [bertvv.bind : Read reverse ipv6 zone hashes] *****************************

TASK [bertvv.bind : Create dict of reverse ipv6 hashes] ************************

TASK [bertvv.bind : Create forward lookup zone file] ***************************
changed: [pr001] => (item=avalon.lan)

TASK [bertvv.bind : Create reverse lookup zone file] ***************************
changed: [pr001] => (item=203.0.113)
changed: [pr001] => (item=172.16)

TASK [bertvv.bind : Create reverse IPv6 lookup zone file] **********************

TASK [bertvv.bind : Main BIND config file] *************************************
changed: [pr001]

TASK [bertvv.bind : Start BIND service] ****************************************
changed: [pr001]

RUNNING HANDLER [bertvv.rh-base : restart journald] ****************************
changed: [pr001]

RUNNING HANDLER [bertvv.rh-base : restart sshd] ********************************
changed: [pr001]

RUNNING HANDLER [bertvv.bind : reload bind] ************************************
changed: [pr001]

PLAY RECAP *********************************************************************
pr001                      : ok=57   changed=28   unreachable=0    failed=0    skipped=18   rescued=0    ignored=0

$ vagrant ssh pr001

This system is built by the Bento project by Chef Software
More information can be found at https://github.com/chef/bento/README.md
             ___   ___  _
 _ __  _ __ / _ \ / _ \/ |
| '_ \| '__| | | | | | | |
| |_) | |  | |_| | |_| | |
| .__/|_|   \___/ \___/|_|
|_|
System information as of Fri Oct 23 13:42:09 UTC 2020.

Distro/Kernel: CentOS Linux 7 (Core) / 3.10.0-957.21.3.el7.x86_64

System load:    1.22, 0.89, 0.38        Memory usage: 21.83% of 991 MiB
System uptime:  4 minutes       Swap   usage: 0.01% of 2047 MiB

Filesystem              Type    Size  Used Avail Use% Mounted on
/dev/mapper/centos-root xfs      44G  1.7G   43G   4% /
/dev/mapper/centos-home xfs      22G   34M   22G   1% /home
/dev/sda1               xfs     1.1G  136M  928M  13% /boot
vagrant                 vboxsf  127G  122G  5.5G  96% /vagrant
total                   -       194G  123G   71G  64% -

Interface       MAC Address             IPv4 Address            IPv6 Address
lo              00:00:00:00:00:00       127.0.0.1/8             ::1/128
eth0            08:00:27:c2:05:d3       10.0.2.15/24            fe80::441b:11ed:d7a3:e089/64
eth1            08:00:27:e5:e3:0f       172.16.128.1/16         fe80::a00:27ff:fee5:e30f/64
[vagrant@pr001 ~]$ sudo /vagrant/test/runbats.sh
Installing BATS
--2020-10-23 13:42:36--  https://github.com/bats-core/bats-core/archive/v1.1.0.tar.gz
Resolving github.com (github.com)... 140.82.121.3
Connecting to github.com (github.com)|140.82.121.3|:443... connected.
HTTP request sent, awaiting response... 302 Found
Location: https://codeload.github.com/bats-core/bats-core/tar.gz/v1.1.0 [following]
--2020-10-23 13:42:36--  https://codeload.github.com/bats-core/bats-core/tar.gz/v1.1.0
Resolving codeload.github.com (codeload.github.com)... 140.82.121.9
Connecting to codeload.github.com (codeload.github.com)|140.82.121.9|:443... connected.
HTTP request sent, awaiting response... 200 OK
Length: unspecified [application/x-gzip]
Saving to: ‘v1.1.0.tar.gz’

    [ <=>                                                                                                                                                                     ] 38,968      --.-K/s   in 0.04s

2020-10-23 13:42:36 (1.03 MB/s) - ‘v1.1.0.tar.gz’ saved [38968]

Installed Bats to /usr/local/bin/bats
Running test /vagrant/test/common.bats
 ✓ SELinux should be set to 'Enforcing'
 ✓ Firewall should be enabled and running
 ✓ EPEL repository should be available
 ✓ Bash-completion should have been installed
 ✓ bind-utils should have been installed
 ✓ Git should have been installed
 ✓ Nano should have been installed
 ✓ Tree should have been installed
 ✓ Vim-enhanced should have been installed
 ✓ Wget should have been installed
 ✓ Admin user maximiliaan should exist
 ✓ An SSH key should have been installed for maximiliaan
 - Custom /etc/motd should have been installed (skipped)

13 tests, 0 failures, 1 skipped
Running test /vagrant/test/pr001/masterdns.bats
 ✓ The dig command should be installed
 ✓ The main config file should be syntactically correct
 ✓ The forward zone file should be syntactically correct
 ✓ The reverse zone files should be syntactically correct
 ✓ The service should be running
 ✓ Forward lookups public servers
 ✓ Forward lookups private servers
 ✓ Reverse lookups public servers
 ✓ Reverse lookups private servers
 ✓ Alias lookups public servers
 ✓ Alias lookups private servers
 ✓ NS record lookup
 ✓ Mail server lookup

13 tests, 0 failures
```

### Test results of `pr002`:

```bash
[vagrant@pr002 ~]$ sudo /vagrant/test/runbats.sh
Running test /vagrant/test/common.bats
 ✓ SELinux should be set to 'Enforcing'
 ✓ Firewall should be enabled and running
 ✓ EPEL repository should be available
 ✓ Bash-completion should have been installed
 ✓ bind-utils should have been installed
 ✓ Git should have been installed
 ✓ Nano should have been installed
 ✓ Tree should have been installed
 ✓ Vim-enhanced should have been installed
 ✓ Wget should have been installed
 ✓ Admin user maximiliaan should exist
 ✓ An SSH key should have been installed for maximiliaan
 - Custom /etc/motd should have been installed (skipped)

13 tests, 0 failures, 1 skipped
Running test /vagrant/test/pr002/slavedns.bats
 ✓ The dig command should be installed
 ✓ The main config file should be syntactically correct
 ✓ The server should be set up as a slave
 ✓ The server should forward requests to the master server
 ✓ There should not be a forward zone file
 ✓ The service should be running
 ✓ Forward lookups public servers
 ✓ Forward lookups private servers
 ✓ Reverse lookups public servers
 ✓ Reverse lookups private servers
 ✓ Alias lookups public servers
 ✓ Alias lookups private servers
 ✓ NS record lookup
 ✓ Mail server lookup

14 tests, 0 failures

```

---

## Resources

* [Dig command](https://www.a2hosting.com/kb/getting-started-guide/internet-and-networking/troubleshooting-dns-with-dig-and-nslookup)
* `ansible\roles\bertvv.bind\README.md`
