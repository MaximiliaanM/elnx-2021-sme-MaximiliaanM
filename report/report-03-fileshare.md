# Enterprise Linux Lab Report

- Student name: Maximiliaan Muylaert
- Github repo: <https://github.com/HoGentTIN/elnx-2021-sme-MaximiliaanM>

---

## Test plan

### Start the vm

Start the vm with `vagrant up pr011`.

### Test `pr011`

SSH into `pr011` with `vagrant ssh pr011`.
Excute the test with `sudo /vagrant/test/runbats.sh`

### Output of test:

```bash
[vagrant@pr011 ~]$ sudo /vagrant/test/runbats.sh
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
Running test /vagrant/test/pr011/samba.bats
 ✓ The ’nmblookup’ command should be installed
 ✓ The ’smbclient’ command should be installed
 ✓ The Samba service should be running
 ✓ The Samba service should be enabled at boot
 ✓ The WinBind service should be running
 ✓ The WinBind service should be enabled at boot
 ✓ The SELinux status should be ‘enforcing’
 ✓ Samba traffic should pass through the firewall
 ✓ Check existence of users
 ✓ Checks shell access of users
 ✓ Samba configuration should be syntactically correct
 ✓ NetBIOS name resolution should work
 ✓ read access for share ‘public’
 ✓ write access for share ‘public’
 ✓ read access for share ‘management’
 ✓ write access for share ‘management’
 ✓ read access for share ‘technical’
 ✓ write access for share ‘technical’
 ✓ read access for share ‘sales’
 ✓ write access for share ‘sales’
 ✓ read access for share ‘it’
 ✓ write access for share ‘it’

22 tests, 0 failures
Running test /vagrant/test/pr011/vsftp.bats
 ✓ VSFTPD service should be running
 ✓ VSFTPD service should be enabled at boot
 ✓ The ’curl’ command should be installed
 ✓ The SELinux status should be ‘enforcing’
 ✓ FTP traffic should pass through the firewall
 ✓ VSFTPD configuration should be syntactically correct
 ✓ Anonymous user should not be able to see shares
 ✓ read access for share ‘public’
 ✓ write access for share ‘public’
 ✓ read access for share ‘management’
 ✓ write access for share ‘management’
 ✓ read access for share ‘technical’
 ✓ write access for share ‘technical’
 ✓ read access for share ‘sales’
 ✓ write access for share ‘sales’
 ✓ read access for share ‘it’
 ✓ write access for share ‘it’

17 tests, 0 failures
```

---

## Procedure/Documentation

### 1. Adding `pr011` as a vagrant host

To set `pr011` as a vagrant host open the `vagrant-hosts.yml` file and enter following code:

```yaml
- name: pr011
  ip: 172.16.128.11
  netmask: 255.255.0.0
```

### 2. Configure playbook & installing roles

Open the playbook `site.yml` and add the host **pr011**. Install the roles `bertvv.samba`, `bertvv.samba` and `bertvv.vsftpd`.

```yaml
- hosts: pr011
  become: true
  roles: 
    - bertvv.rh-base
    - bertvv.samba
    - bertvv.vsftpd
```

The roles are now added to our playbook. To install them, open a bash shell in the main directory and run following script: `./scripts/role-deps.sh.`

### 3. Configure host `pr011`

Make a new file called `pr011.yml` in `ansible\host_vars`. To configure this file follow next steps:

1. Allow samba and vsftpd on the firewall

    ```yaml
    rhbase_allow_services:
      - samba
      - ftp
      - samba-client
    ```

2. Create user groups

    ```yaml
    rhbase_user_groups:
      - management
      - technical
      - sales
      - it
      - public
    ```
3. Create users

    ```yaml
    rhbase_users:
    - name: stevenh
        comment: 'Steven Hermans'
        groups: 
        - management
        - public
        password: '$1$zAiXtpsN$5MNBgTQ/9AYCY.DCSADb50'
        shell: /sbin/nologin
    - name: stevenv
        comment: 'Steven Van Daele'
        groups: 
        - technical
        - public
        password: '$1$dsE8PO0L$0Ud2TLW5r788CiOmutvbJ/'
        shell: /sbin/nologin
    - name: leend
        comment: 'Leen De Meester'
        groups: 
        - technical
        - public
        password: '$1$PUm409Hm$Heo0yeYUrnutc3KbCcVJU0'
        shell: /sbin/nologin
    - name: svena
        comment: 'Sven Aerens'
        groups: 
        - sales
        - public
        password: '$1$yYVHDcjq$KW8G4UD5NRas6Uacg6lbe1'
        shell: /sbin/nologin
    - name: nehirb
        comment: 'Nehir Başar'
        groups: 
        - it
        - public
        password: '$1$gF5l3NVO$HoBhJ6W.A1he4mj40WszO0'
    - name: alexanderd
        comment: 'Alexander De Coninck'
        groups: 
        - technical
        - public
        password: '$1$6qSsjVcU$l82lCUvc2VaSVk1nG8vY81'
        shell: /sbin/nologin
    - name: krisv
        comment: 'Kris Vanhaverbeke'
        groups: 
        - management
        - public
        password: '$1$nCw9oVHo$raOe3TE/pYQM6leszSS54.'
        shell: /sbin/nologin
    - name: benoitp
        comment: 'Benoit Pluquet'
        groups: 
        - sales
        - public
        password: '$1$qvbXL1bd$52FdoVmtzg6neBJvJOxKo1'
        shell: /sbin/nologin
    - name: anc
        comment: 'An Coppens'
        groups: 
        - technical
        - public
        password: '$1$YD75/B/T$x1OjpJbYb8rVpmDHQ3lOq0'
        shell: /sbin/nologin
    - name: elenaa
        comment: 'Elena Andreev'
        groups: 
        - management
        - public
        password: '$1$P7q8K69h$ZINIDvDZCkqpkGqURSRU//'
        shell: /sbin/nologin
    - name: evyt
        comment: 'Evy Tilleman'
        groups: 
        - management
        - public
        password: '$1$fcryQT7R$zXlfCyNdnO5UDaB6/fTcx1'
        shell: /sbin/nologin
    - name: christophev
        comment: 'Christophe Van der Meersche'
        groups: 
        - it
        - public
        password: '$1$387i/2QK$Fo8dsKlwJBup5ktE0apAK1'
    - name: stefaanv
        comment: 'Stefaan Vernimmen'
        groups: 
        - technical
        - public
        password: '$1$DlJMq3VR$hkgVRWPELVkVTPja.a/7z0'
        shell: /sbin/nologin
    - name: maximiliaan
        comment: 'Maximiliaan Muylaert'
        groups: 
        - wheel
        - it
        - public
        password: '$6$6QhrgSAjVwd2IWk8$pzPgb8nirer5NNCupmzei9jMZp5MjuQfkgLuqahI8SYp1yh93./pEHhT9Dp6rjqqhf.rTdfGyAvgsBtO0PYeP1'
    ```

4. Create samba users

    ```yaml
    samba_users:
      - name: stevenh
        password: stevenh
      - name: stevenv
        password: stevenv
      - name: leend
        password: leend
      - name: svena
        password: svena
      - name: nehirb
        password: nehirb
      - name: alexanderd
        password: alexanderd
      - name: krisv
        password: krisv
      - name: benoitp
        password: benoitp
      - name: anc
        password: anc
      - name: elenaa
        password: elenaa
      - name: evyt
        password: evyt
      - name: christophev
        password: christophev
      - name: stefaanv
        password: stefaanv
      - name: maximiliaan
        password: maximiliaan
    ```

5. Create samba_shares

    ```yaml
    samba_shares:
    - name: management
        comment: 'Management - inaccessible outside this unit'
        group: management
        browseable: yes
        writable: yes
        write_list: +management
        valid_users: +management
        create_mode: 0770
        directory_mode: 0770
        path: /srv/shares/management
    - name: technical
        comment: 'Technical - visible outside this unit, not writable'
        group: technical
        browseable: yes
        writable: yes
        write_list: +technical
        valid_users: +technical, +public
        create_mode: "0775"
        directory_mode: "0775"
        path: /srv/shares/technical
    - name: sales
        comment: 'Sales - visible to management, not writable'
        group: sales
        browseable: yes
        writable: yes
        write_list: +sales
        valid_users: +sales, +management
        create_mode: 0770
        directory_mode: 0770
        path: /srv/shares/sales
    - name: it
        comment: 'IT - visible to management, not writable'
        group: it
        browseable: yes
        writable: yes
        write_list: +it
        valid_users: +it, +management
        create_mode: 0770
        directory_mode: 0770
        path: /srv/shares/it
    - name: public
        comment: 'public'
        group: public
        browseable: yes
        writable: yes
        write_list: +public
        valid_users: +public
        create_mode: "0775"
        directory_mode: "0775"
        path: /srv/shares/sales
    ```

6. Make the share `it` and `sales` visible to management (not writable)

    ```yaml
    vsftpd_extra_permissions:
    - folder: "/srv/shares/sales"
        entity: "management"
        etype: "group"
        permissions: "r-x"
    - folder: "/srv/shares/it"
        entity: "management"
        etype: "group"
        permissions: "r-x"
    ```

7. Disable anonymous browsing and enable login

    ```yaml
    vsftpd_anonymous_enable: false
    vsftpd_local_enable: true
    vsftpd_local_root: /srv/shares
    ```

---

## Test report

After running all tests, only the `vsftpd.bats` test failed. I had following result:

```bash
Running test /vagrant/test/pr011/vsftp.bats
 ✓ VSFTPD service should be running
 ✓ VSFTPD service should be enabled at boot
 ✓ The ’curl’ command should be installed
 ✓ The SELinux status should be ‘enforcing’
 ✓ FTP traffic should pass through the firewall
 ✓ VSFTPD configuration should be syntactically correct
 ✓ Anonymous user should not be able to see shares
 ✗ read access for share ‘public’
   (from function `assert_read_access' in file /vagrant/test/pr011/vsftp.bats, line 37,
    in test file /vagrant/test/pr011/vsftp.bats, line 135)
     `assert_read_access     public     alexanderd    alexanderd' failed with status 67
 ✗ write access for share ‘public’
   (from function `assert_write_access' in file /vagrant/test/pr011/vsftp.bats, line 61,
    in test file /vagrant/test/pr011/vsftp.bats, line 153)
     `assert_write_access    public     alexanderd    alexanderd' failed with status 67
 ✗ read access for share ‘management’
   (from function `assert_no_read_access' in file /vagrant/test/pr011/vsftp.bats, line 48,
    in test file /vagrant/test/pr011/vsftp.bats, line 171)
     `assert_no_read_access  management alexanderd    alexanderd' failed with status 67
 ✗ write access for share ‘management’
   (from function `assert_write_access' in file /vagrant/test/pr011/vsftp.bats, line 61,
    in test file /vagrant/test/pr011/vsftp.bats, line 193)
     `assert_write_access    management elenaa        elenaa' failed with status 67
 ✗ read access for share ‘technical’
   (from function `assert_read_access' in file /vagrant/test/pr011/vsftp.bats, line 37,
    in test file /vagrant/test/pr011/vsftp.bats, line 207)
     `assert_read_access     technical  alexanderd    alexanderd' failed with status 67
 ✗ write access for share ‘technical’
   (from function `assert_write_access' in file /vagrant/test/pr011/vsftp.bats, line 61,
    in test file /vagrant/test/pr011/vsftp.bats, line 225)
     `assert_write_access    technical  alexanderd    alexanderd' failed with status 67
 ✗ read access for share ‘sales’
   (from function `assert_no_read_access' in file /vagrant/test/pr011/vsftp.bats, line 48,
    in test file /vagrant/test/pr011/vsftp.bats, line 243)
     `assert_no_read_access  sales      alexanderd    alexanderd' failed with status 67
 ✗ write access for share ‘sales’
   (from function `assert_write_access' in file /vagrant/test/pr011/vsftp.bats, line 61,
    in test file /vagrant/test/pr011/vsftp.bats, line 263)
     `assert_write_access    sales      benoitp       benoitp' failed with status 67
 ✗ read access for share ‘it’
   (from function `assert_no_read_access' in file /vagrant/test/pr011/vsftp.bats, line 48,
    in test file /vagrant/test/pr011/vsftp.bats, line 279)
     `assert_no_read_access  it         alexanderd    alexanderd' failed with status 67
 ✗ write access for share ‘it’
   (from function `assert_write_access' in file /vagrant/test/pr011/vsftp.bats, line 61,
    in test file /vagrant/test/pr011/vsftp.bats, line 310)
     `assert_write_access    it         ${admin_user} ${admin_password}' failed with status 67
   curl: (19) RETR response: 257
   curl: (19) RETR response: 257

17 tests, 10 failures

```

After some debugging I found out that the test failed with status 67 which corresponds with no login for users. I searched the web and found [a fix](https://serverfault.com/questions/358324/ftp-doesnt-allow-usr-sbin-nologin-user) for this bug. I added a pre-task in the playbook `site.yaml`.

```yaml
pre_tasks:
    - name: Add /sbin/nologin to /etc/shells. Allows local users to access ftp without shell access
      lineinfile:
        path: /etc/shells
        line: /sbin/nologin
```

This allows users to login and not have permission to access the shell.

After running the tests again, every test passed.

```bash
[vagrant@pr011 ~]$ sudo /vagrant/test/runbats.sh
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
Running test /vagrant/test/pr011/samba.bats
 ✓ The ’nmblookup’ command should be installed
 ✓ The ’smbclient’ command should be installed
 ✓ The Samba service should be running
 ✓ The Samba service should be enabled at boot
 ✓ The WinBind service should be running
 ✓ The WinBind service should be enabled at boot
 ✓ The SELinux status should be ‘enforcing’
 ✓ Samba traffic should pass through the firewall
 ✓ Check existence of users
 ✓ Checks shell access of users
 ✓ Samba configuration should be syntactically correct
 ✓ NetBIOS name resolution should work
 ✓ read access for share ‘public’
 ✓ write access for share ‘public’
 ✓ read access for share ‘management’
 ✓ write access for share ‘management’
 ✓ read access for share ‘technical’
 ✓ write access for share ‘technical’
 ✓ read access for share ‘sales’
 ✓ write access for share ‘sales’
 ✓ read access for share ‘it’
 ✓ write access for share ‘it’

22 tests, 0 failures
Running test /vagrant/test/pr011/vsftp.bats
 ✓ VSFTPD service should be running
 ✓ VSFTPD service should be enabled at boot
 ✓ The ’curl’ command should be installed
 ✓ The SELinux status should be ‘enforcing’
 ✓ FTP traffic should pass through the firewall
 ✓ VSFTPD configuration should be syntactically correct
 ✓ Anonymous user should not be able to see shares
 ✓ read access for share ‘public’
 ✓ write access for share ‘public’
 ✓ read access for share ‘management’
 ✓ write access for share ‘management’
 ✓ read access for share ‘technical’
 ✓ write access for share ‘technical’
 ✓ read access for share ‘sales’
 ✓ write access for share ‘sales’
 ✓ read access for share ‘it’
 ✓ write access for share ‘it’

17 tests, 0 failures
```

---

## Resources

* [Samba configuration](https://www.samba.org/samba/docs/man/manpages-3/smb.conf.5.html)
* [Test playbook](https://github.com/bertvv/ansible-role-samba/blob/docker-tests/test.yml)
* [Linux permission calculator](http://permissions-calculator.org/)
* [/sbin/nologin fix](https://serverfault.com/questions/358324/ftp-doesnt-allow-usr-sbin-nologin-user)
* README files of the roles samba and vsftpd
