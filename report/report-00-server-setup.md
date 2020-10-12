# Enterprise Linux Lab Report - Server Setup

- Student name: Maximiliaan Muylaert
- Github repo: <https://github.com/HoGentTIN/elnx-2021-sme-MaximiliaanM>

Describe the goals of the current iteration/assignment in a short sentence.

---

## Test plan

1. On the host system, go to the local working directory of the project repository
2. Execute `vagrant status`
    - There should be one VM, `pu001` with status `not created`. If the VM does exist, destroy it first with `vagrant destroy -f pu001`
3. Execute `vagrant up pu001`
    - The command should run without errors (exit status 0)
4. Log in on the server with `vagrant ssh pu001` and run the acceptance tests. They should succeed

    ```console
    [vagrant@pu001 test]$ sudo /vagrant/test/runbats.sh
    Running test /vagrant/test/common.bats
    ✓ EPEL repository should be available
    ✓ Bash-completion should have been installed
    ✓ bind-utils should have been installed
    ✓ Git should have been installed
    ✓ Nano should have been installed
    ✓ Tree should have been installed
    ✓ Vim-enhanced should have been installed
    ✓ Wget should have been installed
    ✓ Admin user bert should exist
    ✓ Custom /etc/motd should be installed

    10 tests, 0 failures
    ```

    Any tests for the LAMP stack may fail, but this is not part of the current assignment.

5. Log off from the server and ssh to the VM as described below. You should **not** get a password prompt.

    ```console
    $ ssh maximiliaan@203.0.113.10
    Welcome to pu001.localdomain.
    enp0s3     : 10.0.2.15         fe80::a00:27ff:fe5c:6428/64
    enp0s8     : 203.0.113.10      fe80::a00:27ff:fecd:aeed/64
    [maximiliaan@pu001 ~]$
    ```

---

## Procedure/Documentation

1. Add the role `bertvv.rh-base` to the roles in **site.yaml**.

2. Create in the folder ansible a new folder called **host_vars**.

3. In the newly created folder, make a new new file called **pu001.yml**.

4. To enable firewall enter:
    ```yaml
    rhbase_firewall_allow_services:
    - https
    - http
    ```
5. To set SELinux to enforcing enter:
    ```yaml
    rhbase_selinux_state: 'enforcing'
    ```
6. To install the EPEL repository enter:
    ```yaml
    rhbase_repositories:
    - epel-release
    ```
7. Open the file **all.yml** (located in `ansible\group_vars\all.yml`)

8. To install additional packages on all servers enter:
    ```yaml
    rhbase_install_packages:
    - bash-completion
    - bind-utils
    - git
    - nano
    - tree
    - vim-enhanced
    - wget
    ```
9. To create a user account as an administrator enter following code:
    ```yaml
    rhbase_users:
    - name: 
      comment: 'Administrator'
      groups:
        - wheel
      password: 
      ssh_key:
    ```
    * Use your own name in lowercase for **name**.
    * Add wheel to **groups** to make a user an administrator.
    * The password should be specified as a hash, as returned by [crypt(3)](http://man7.org/linux/man-pages/man3/crypt.3.html)
    * Generate a SSH key on the host system with `ssh-keygen `
10. To allow vagrant to use SSH enter:
    ```yaml
    rhbase_ssh_allow_groups:
      - vagrant
    ```
11. To use the option for generating a custom `/etc/motd` file enter:
    ```yaml
    rhbase_dynamic_motd: true
    ```

---

## Test report

1. Local working directory found
    ```bash
    Maximiliaan@MSI MINGW64 ~
    $ cd ~/Documents/School/3de\ jaar/EnterpriseLinux/elnx-2021-sme-MaximiliaanM/
    ```

2. Result of `vagrant status`:
    ```bash
    $ vagrant status
    Current machine states:

    router                    not created (virtualbox)
    pu001                     not created (virtualbox)

    This environment represents multiple VMs. The VMs are all listed
    above with their current state. For more information about a specific
    VM, run `vagrant status NAME`.

    ```

3. Result of `vagrant up pu001`:
    ```bash
    $ vagrant up pu001
    ==> vagrant: A new version of Vagrant is available: 2.2.10 (installed version: 2.2.7)!
    ==> vagrant: To upgrade visit: https://www.vagrantup.com/downloads.html

    Bringing machine 'pu001' up with 'virtualbox' provider...
    ==> pu001: Importing base box 'bento/centos-8.2'...
    ==> pu001: Matching MAC address for NAT networking...
    ==> pu001: Checking if box 'bento/centos-8.2' version '202008.16.0' is up to date...
    ==> pu001: Setting the name of the VM: elnx-2021-sme-MaximiliaanM_pu001_1602524721749_97423
    ==> pu001: Clearing any previously set network interfaces...
    ==> pu001: Preparing network interfaces based on configuration...
        pu001: Adapter 1: nat
        pu001: Adapter 2: hostonly
    ==> pu001: Forwarding ports...
        pu001: 22 (guest) => 2222 (host) (adapter 1)
    ==> pu001: Running 'pre-boot' VM customizations...
    ==> pu001: Booting VM...
    ==> pu001: Waiting for machine to boot. This may take a few minutes...
        pu001: SSH address: 127.0.0.1:2222
        pu001: SSH username: vagrant
        pu001: SSH auth method: private key
        pu001:
        pu001: Vagrant insecure key detected. Vagrant will automatically replace
        pu001: this with a newly generated keypair for better security.
        pu001:
        pu001: Inserting generated public key within guest...
    ==> pu001: Machine booted and ready!
    GuestAdditions are newer than your host but, downgrades are disabled. Skipping.
    ==> pu001: Checking for guest additions in VM...
        pu001: The guest additions on this VM do not match the installed version of
        pu001: VirtualBox! In most cases this is fine, but in rare cases it can
        pu001: prevent things such as shared folders from working properly. If you see
        pu001: shared folder errors, please make sure the guest additions within the
        pu001: virtual machine match the version of VirtualBox you have installed on
        pu001: your host and reload your VM.
        pu001:
        pu001: Guest Additions Version: 6.1.12
        pu001: VirtualBox Version: 6.0
    ==> pu001: Setting hostname...
    ==> pu001: Configuring and enabling network interfaces...
    ==> pu001: Mounting shared folders...
        pu001: /vagrant => C:/Users/Maximiliaan/Documents/School/3de jaar/EnterpriseLinux/elnx-2021-sme-MaximiliaanM
    ==> pu001: Running provisioner: ansible_local...
        pu001: Installing Ansible...
        pu001: Running ansible-playbook...

    PLAY [pu001] *******************************************************************

    TASK [Gathering Facts] *********************************************************
    ok: [pu001]

    TASK [bertvv.rh-base : Load distro specific variables] *************************
    ok: [pu001] => (item=/vagrant/ansible/roles/bertvv.rh-base/vars/RedHat-8.yml)

    TASK [bertvv.rh-base : include_tasks] ******************************************
    included: /vagrant/ansible/roles/bertvv.rh-base/tasks/install.yml for pu001

    TASK [bertvv.rh-base : Install | Check minimal value of ‘rhbase_repo_installonly_limit’ (>= 2)] ***
    ok: [pu001] => {
        "msg": "The value of ‘rhbase_repo_installonly_limit’ should be at least 2, actual value is 3."
    }

    TASK [bertvv.rh-base : Install | Ensure the machine-ID is available] ***********
    ok: [pu001]

    TASK [bertvv.rh-base : Install | Ensure basic systemd services are running] ****
    ok: [pu001] => (item=systemd-journald)
    ok: [pu001] => (item=systemd-tmpfiles-setup-dev)
    ok: [pu001] => (item=systemd-tmpfiles-setup)

    TASK [bertvv.rh-base : Install | Role/Ansible dependencies] ********************
    ok: [pu001]

    TASK [bertvv.rh-base : Install | Package management configuration (dnf)] *******
    changed: [pu001]

    TASK [bertvv.rh-base : Install | Ensure specified external repositories are installed] ***
    ok: [pu001]

    TASK [bertvv.rh-base : Install | Ensure specified repositories are enabled] ****

    TASK [bertvv.rh-base : Install | Ensure specified packages are installed] ******
    changed: [pu001]

    TASK [bertvv.rh-base : Install | Ensure specified packages are NOT installed] ***
    ok: [pu001]

    TASK [bertvv.rh-base : Install | Ensure all updates are installed] *************
    skipping: [pu001]

    TASK [bertvv.rh-base : include_tasks] ******************************************
    included: /vagrant/ansible/roles/bertvv.rh-base/tasks/config.yml for pu001

    TASK [bertvv.rh-base : Config | Ensure host name is in the hosts file] *********
    skipping: [pu001]

    TASK [bertvv.rh-base : Config | Install dependency for dynamic MotD] ***********
    changed: [pu001]

    TASK [bertvv.rh-base : Config | Install script generating dynamic MotD] ********
    changed: [pu001]

    TASK [bertvv.rh-base : Config | Check if ifup-eth script overrides firewall zone] ***
    ok: [pu001]

    TASK [bertvv.rh-base : Config | Don’t override firewall zone in ifup-eth script] ***
    skipping: [pu001]

    TASK [bertvv.rh-base : Config | Check if ifup-post script overrides firewall zone] ***
    ok: [pu001]

    TASK [bertvv.rh-base : Config | Don’t override firewall zone in ifup-post script] ***
    skipping: [pu001]

    TASK [bertvv.rh-base : Config | Set the TZ environment variable] ***************
    changed: [pu001]

    TASK [bertvv.rh-base : Config | Set protocol version for SSH] ******************
    changed: [pu001]

    TASK [bertvv.rh-base : Config | Set PermitEmptyPasswords] **********************
    changed: [pu001]

    TASK [bertvv.rh-base : Config | Set IgnoreRhosts] ******************************
    changed: [pu001]

    TASK [bertvv.rh-base : Config | Set HostbasedAuthentication] *******************
    changed: [pu001]

    TASK [bertvv.rh-base : Config | Set RhostsRSAAuthentication] *******************
    changed: [pu001]

    TASK [bertvv.rh-base : include_tasks] ******************************************
    included: /vagrant/ansible/roles/bertvv.rh-base/tasks/services.yml for pu001

    TASK [bertvv.rh-base : Services | Ensure SSH daemon is running] ****************
    ok: [pu001]

    TASK [bertvv.rh-base : Services | Ensure `/var/log/journal` exists] ************
    changed: [pu001]

    TASK [bertvv.rh-base : Services | Ensure specified services are running] *******

    TASK [bertvv.rh-base : Services | Ensure specified services are NOT running] ***

    TASK [bertvv.rh-base : include_tasks] ******************************************
    included: /vagrant/ansible/roles/bertvv.rh-base/tasks/security.yml for pu001

    TASK [bertvv.rh-base : Security | Make sure SELinux has the desired state (enforcing)] ***
    changed: [pu001]

    TASK [bertvv.rh-base : Security | Enable SELinux booleans] *********************

    TASK [bertvv.rh-base : Security | Make sure the firewall is running] ***********
    changed: [pu001]

    TASK [bertvv.rh-base : Security | Make sure basic services can pass through firewall] ***
    ok: [pu001] => (item=dhcpv6-client)
    ok: [pu001] => (item=ssh)

    TASK [bertvv.rh-base : Security | Make sure user specified services can pass through firewall] ***
    changed: [pu001] => (item=https)
    changed: [pu001] => (item=http)

    TASK [bertvv.rh-base : Security | Make sure user specified ports can pass through firewall] ***

    TASK [bertvv.rh-base : Security | Make sure specified interfaces are added] ****

    TASK [bertvv.rh-base : include_tasks] ******************************************
    included: /vagrant/ansible/roles/bertvv.rh-base/tasks/users.yml for pu001

    TASK [bertvv.rh-base : Users | Process groups] *********************************
    ok: [pu001] => (item={'name': 'maximiliaan', 'comment': 'Administrator', 'groups': ['wheel'], 'password': '$6$6QhrgSAjVwd2IWk8$pzPgb8nirer5NNCupmzei9jMZp5MjuQfkgLuqahI8SYp1yh93./pEHhT9Dp6rjqqhf.rTdfGyAvgsBtO0PYeP1', 'ssh_key': 'ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAACAQDSKv3fs5Mg0SV33eS+ho96xEHByU3SmI0/UMmXmFkmqWJf+TQO44nzNyOwZiAmLXtZnle5QI92X9GfCm77iTEYaye4wLy60NVoyxg+CggOhaIcc6IxBOajV8YoGHVG7/ZwiUXx/b/pzAcJScXe7n+DIxSkyhJ65r6WIoZhjE5phOsKopjRRJwvsfQsSOHapjeOZuSyJD7T4m1PBvrAnGQ6607mF05fG6UPHCdH3zwMnuutzdLllc4FmmvlqRv6FNAziEouZYenHjRMNOoBwW7fK7RiZrECZnaiqyvY3a+PgKXCLcaEcsdYBdjiJFnZtEITuXVJ3kqZdsJNYbn/bhFK29PsywVC9h1+RI5vJy0DGu1FTH5viAYDeHkm4Us7rAwjsplrHEuGLLfpXPlg8FBHPa/LBBVqkcTY/RWyQyWjUUQKq1eSJgzQvfTuKh9RSyszMOERsJ75fLJlT9aSMNdfXeeOfdHZagMGVPhuEmdYEA1rPNUBV7uBsiC+AS8Ip/Ee4+XvnLRALz4cfS8SN/v344uHORKIAtXsdXAI4YIuunBbiA1dY6x2+V4kHBveMgqtnUKwud++O1ADJH/C1CGHEk0cZeshDIIdnUlsTOMieFe3wrATnGP3Rp5sK0j09jn/YxDsHlkeSA9F66oAZxBynyEX9xduYMpRJ+YShMHFeQ== Maximiliaan@MSI'})

    TASK [bertvv.rh-base : Users | Add groups] *************************************
    ok: [pu001] => (item=wheel)

    TASK [bertvv.rh-base : Users | Add users] **************************************
    changed: [pu001] => (item={'name': 'maximiliaan', 'comment': 'Administrator', 'groups': ['wheel'], 'password': '$6$6QhrgSAjVwd2IWk8$pzPgb8nirer5NNCupmzei9jMZp5MjuQfkgLuqahI8SYp1yh93./pEHhT9Dp6rjqqhf.rTdfGyAvgsBtO0PYeP1', 'ssh_key': 'ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAACAQDSKv3fs5Mg0SV33eS+ho96xEHByU3SmI0/UMmXmFkmqWJf+TQO44nzNyOwZiAmLXtZnle5QI92X9GfCm77iTEYaye4wLy60NVoyxg+CggOhaIcc6IxBOajV8YoGHVG7/ZwiUXx/b/pzAcJScXe7n+DIxSkyhJ65r6WIoZhjE5phOsKopjRRJwvsfQsSOHapjeOZuSyJD7T4m1PBvrAnGQ6607mF05fG6UPHCdH3zwMnuutzdLllc4FmmvlqRv6FNAziEouZYenHjRMNOoBwW7fK7RiZrECZnaiqyvY3a+PgKXCLcaEcsdYBdjiJFnZtEITuXVJ3kqZdsJNYbn/bhFK29PsywVC9h1+RI5vJy0DGu1FTH5viAYDeHkm4Us7rAwjsplrHEuGLLfpXPlg8FBHPa/LBBVqkcTY/RWyQyWjUUQKq1eSJgzQvfTuKh9RSyszMOERsJ75fLJlT9aSMNdfXeeOfdHZagMGVPhuEmdYEA1rPNUBV7uBsiC+AS8Ip/Ee4+XvnLRALz4cfS8SN/v344uHORKIAtXsdXAI4YIuunBbiA1dY6x2+V4kHBveMgqtnUKwud++O1ADJH/C1CGHEk0cZeshDIIdnUlsTOMieFe3wrATnGP3Rp5sK0j09jn/YxDsHlkeSA9F66oAZxBynyEX9xduYMpRJ+YShMHFeQ== Maximiliaan@MSI'})

    TASK [bertvv.rh-base : Users | Set up SSH keys] ********************************
    changed: [pu001] => (item={'name': 'maximiliaan', 'comment': 'Administrator', 'groups': ['wheel'], 'password': '$6$6QhrgSAjVwd2IWk8$pzPgb8nirer5NNCupmzei9jMZp5MjuQfkgLuqahI8SYp1yh93./pEHhT9Dp6rjqqhf.rTdfGyAvgsBtO0PYeP1', 'ssh_key': 'ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAACAQDSKv3fs5Mg0SV33eS+ho96xEHByU3SmI0/UMmXmFkmqWJf+TQO44nzNyOwZiAmLXtZnle5QI92X9GfCm77iTEYaye4wLy60NVoyxg+CggOhaIcc6IxBOajV8YoGHVG7/ZwiUXx/b/pzAcJScXe7n+DIxSkyhJ65r6WIoZhjE5phOsKopjRRJwvsfQsSOHapjeOZuSyJD7T4m1PBvrAnGQ6607mF05fG6UPHCdH3zwMnuutzdLllc4FmmvlqRv6FNAziEouZYenHjRMNOoBwW7fK7RiZrECZnaiqyvY3a+PgKXCLcaEcsdYBdjiJFnZtEITuXVJ3kqZdsJNYbn/bhFK29PsywVC9h1+RI5vJy0DGu1FTH5viAYDeHkm4Us7rAwjsplrHEuGLLfpXPlg8FBHPa/LBBVqkcTY/RWyQyWjUUQKq1eSJgzQvfTuKh9RSyszMOERsJ75fLJlT9aSMNdfXeeOfdHZagMGVPhuEmdYEA1rPNUBV7uBsiC+AS8Ip/Ee4+XvnLRALz4cfS8SN/v344uHORKIAtXsdXAI4YIuunBbiA1dY6x2+V4kHBveMgqtnUKwud++O1ADJH/C1CGHEk0cZeshDIIdnUlsTOMieFe3wrATnGP3Rp5sK0j09jn/YxDsHlkeSA9F66oAZxBynyEX9xduYMpRJ+YShMHFeQ== Maximiliaan@MSI'})

    TASK [bertvv.rh-base : include_tasks] ******************************************
    included: /vagrant/ansible/roles/bertvv.rh-base/tasks/admin.yml for pu001

    TASK [bertvv.rh-base : Admin | Make sure users from the wheel group can use sudo] ***
    changed: [pu001]

    TASK [bertvv.rh-base : Admin | Set attributes of sudo configuration file for wheel group] ***
    changed: [pu001]

    TASK [bertvv.rh-base : Admin | Make sure only these groups can ssh] ************
    changed: [pu001]

    TASK [bertvv.rh-base : include_tasks] ******************************************
    skipping: [pu001]

    RUNNING HANDLER [bertvv.rh-base : restart journald] ****************************
    changed: [pu001]

    RUNNING HANDLER [bertvv.rh-base : restart firewalld] ***************************
    changed: [pu001]

    RUNNING HANDLER [bertvv.rh-base : restart sshd] ********************************
    changed: [pu001]

    PLAY RECAP *********************************************************************
    pu001                      : ok=42   changed=22   unreachable=0    failed=0    skipped=11   rescued=0    ignored=0
    ```

4. Result of `vagrant ssh pu001` and running acceptance test:
    ```bash
    $ vagrant ssh pu001

    This system is built by the Bento project by Chef Software
    More information can be found at https://github.com/chef/bento
                 '___   ___  _
     _ __  _   _ / _ \ / _ \/ |
    |  _ \| | | | | | | | | | |
    | |_) | |_| | |_| | |_| | |
    | .__/ \__,_|\___/ \___/|_|
    |_|'
    System information as of Mon Oct 12 17:58:26 UTC 2020.

    Distro/Kernel: CentOS Linux 8 (Core) / 4.18.0-193.14.2.el8_2.x86_64

    System load:    0.00, 0.12, 0.17        Memory usage: 23.44% of 818 MiB
    System uptime:  12 minutes      Swap   usage: 0.09% of 2215 MiB

    Filesystem          Type    Size  Used Avail Use% Mounted on
    /dev/mapper/cl-root xfs      44G  1.7G   43G   4% /
    /dev/mapper/cl-home xfs      22G  184M   22G   1% /home
    /dev/sda1           ext4    1.1G  169M  785M  18% /boot
    vagrant             vboxsf  127G  117G  9.8G  93% /vagrant
    total               -       193G  119G   74G  62% -

    Interface       MAC Address             IPv4 Address            IPv6 Address
    lo              00:00:00:00:00:00       127.0.0.1/8             ::1/128
    eth0            08:00:27:b6:a4:8f       10.0.2.15/24            fe80::5081:470e:1f98:f656/64
    eth1            08:00:27:e3:3c:3d       203.0.113.10/24         fe80::a00:27ff:fee3:3c3d/64

    [vagrant@pu001 ~]$ sudo /vagrant/test/runbats.sh
    Installing BATS
    --2020-10-12 17:59:28--  https://github.com/bats-core/bats-core/archive/v1.1.0.tar.gz
    Resolving github.com (github.com)... 140.82.121.3
    Connecting to github.com (github.com)|140.82.121.3|:443... connected.
    HTTP request sent, awaiting response... 302 Found
    Location: https://codeload.github.com/bats-core/bats-core/tar.gz/v1.1.0 [following]
    --2020-10-12 17:59:28--  https://codeload.github.com/bats-core/bats-core/tar.gz/v1.1.0
    Resolving codeload.github.com (codeload.github.com)... 140.82.121.10
    Connecting to codeload.github.com (codeload.github.com)|140.82.121.10|:443... connected.
    HTTP request sent, awaiting response... 200 OK
    Length: unspecified [application/x-gzip]
    Saving to: ‘v1.1.0.tar.gz’

    v1.1.0.tar.gz                                            [ <=>                                                                                                                 ]  38.05K  --.-KB/s    in 0.03s

    2020-10-12 17:59:28 (1.25 MB/s) - ‘v1.1.0.tar.gz’ saved [38968]

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
    ✗ Admin user bert should exist
    (in test file /vagrant/test/common.bats, line 51)
        `getent passwd ${admin_user}' failed with status 2
    ✗ An SSH key should have been installed for bert
    (in test file /vagrant/test/common.bats, line 58)
        `[ -f "${keyfile}" ]' failed
    - Custom /etc/motd should have been installed (skipped)

    13 tests, 2 failures, 1 skipped
    ```
    Forgot to change user ***bert*** in `common.bats`
    ```bash
    admin_user=maximiliaan
    ```
    After change:
    ```bash
    [vagrant@pu001 test]$ sudo /vagrant/test/runbats.sh
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
    ```

5. Result of other SSH method:
    ```bash
    $ ssh maximiliaan@203.0.113.10
    The authenticity of host '203.0.113.10 (203.0.113.10)' can't be established.
    ECDSA key fingerprint is SHA256:GTvYvtGBkiqIdw1Drj99wgiE6vVg4qP+EUnCxQDEbZw.
    Are you sure you want to continue connecting (yes/no)? yes
    Warning: Permanently added '203.0.113.10' (ECDSA) to the list of known hosts.
    maximiliaan@203.0.113.10's password:

    ```
    The vm still asks me to enter my password. This is due to the fact that I created a new SSH-key for this assignment which differs from my personal SSH-key for github. I don't use my personal SSH-key for security reasons.

---

## Resources

* Sylabus
* README in `ansible\roles\bertvv.rh-base\README.md`
* [SSH-keygen](https://confluence.atlassian.com/bitbucketserver/creating-ssh-keys-776639788.html)