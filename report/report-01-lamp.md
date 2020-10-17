# Enterprise Linux Lab Report

- Student name: Maximiliaan Muylaert
- Github repo: <https://github.com/HoGentTIN/elnx-2021-sme-MaximiliaanM>

---

## Test plan
In the test script `test/pu001/lamp.bats`, you may want to change the variables in the test script to the values you have used in your configuration script:

```bash
mariadb_root_password=fogMeHud8
wordpress_database=wp_db
wordpress_user=wp_user
wordpress_password=CorkIg
```
1. Start the VM with `vagrant up pu001` in a bash shell.

2. Log in into the VM with `vagrant ssh pu001`.

3. Run the test file with `sudo /vagrant/test/runbats.sh`. You should get following output:

```bash
[vagrant@pu001 /]$ sudo /vagrant/test/runbats.sh
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
Running test /vagrant/test/pu001/lamp.bats
 ✓ The necessary packages should be installed
 ✓ The Apache service should be running
 ✓ The Apache service should be started at boot
 ✓ The MariaDB service should be running
 ✓ The MariaDB service should be started at boot
 ✓ The SELinux status should be ‘enforcing’
 ✓ Web traffic should pass through the firewall
 ✓ Mariadb should have a database for Wordpress
 ✓ The MariaDB user should have write
 ✓ The website should be accessible through HTTP
 ✓ The website should be accessible through HTTPS
 ✓ The certificate should not be the default one
 ✓ The Wordpress install page should be visible under http://203.0.113.10/wordpress/
 ✓ MariaDB should not have a test database
 ✓ MariaDB should not have anonymous users

15 tests, 0 failures
```

---

## Procedure/Documentation

### 1. Adding roles

Add the necessary roles for the LAMP stack in `ansible\site.yml` that you can find on <https://galaxy.ansible.com/bertvv/>.</br> It should look like this:

```yaml
- hosts: pu001
  become: true
  roles:
    - bertvv.rh-base
    - bertvv.httpd
    - bertvv.mariadb
    - bertvv.wordpress
```

After adding them to the `ansible\site.yml` file, we're going to install them. Open a bash shell in the main directory and run following script: `./scripts/role-deps.sh`.

### 2. Activate SELinux and enable the firewall

SELinux and the firewall was already configured in the previous assignment but if this is not the case, you can insert following code to `ansible\host_vars\pu001.yml`.

```yaml
rhbase_firewall_allow_services:
  - https
  - http

rhbase_selinux_state: 'enforcing'
```

### 3. Configure MariaDB

To configure MariaDB on our host `pu001` enter following code:

```yaml
mariadb_mirror: 'mariadb.mirror.nucleus.be/yum'

mariadb_root_password: <INSERT_ROOT_PASSWORD>

mariadb_databases: 
  - name: <INSERT_DATABASE_NAME>

mariadb_users:
  - name: <INSERT_USERNAME>
    password: <INSERT_USER_PASSWORD>
    priv: '*.*:ALL,GRANT'
```

* `mariadb_mirror`: Specifies custom download mirror to speed up download process.
* `mariadb_root_password`: Change the root password. (**highly recommended!**)
* `mariadb_databases`: Specifies the name of the database.
* `mariadb_users`: Configure users
    * `name`: name of user.
    * `password`: password of user
    * `priv`: Specifies the privileges for the user. [more info in Ansible documentation](https://docs.ansible.com/ansible/latest/collections/community/mysql/mysql_user_module.html).

**NOTE** that you need to change the `<INSERT>` variables to your own usernames and passwords.

### 4. Configure Wordpress

To configure Wordpress on our host `pu001` enter following code:

```yaml
wordpress_database: Wordpress_DB

wordpress_user: <INSERT_USERNAME>

wordpress_password: <INSERT_USER_PASSWORD>
```
* `wordpress_database`: Specifies which database to use.
* `wordpress_user`: Username of the Wordpress user. Used to login on the admin dashboard
* `wordpress_password`: Password of the Wordpress user. Used to login on the admin dashboard

**NOTE** that you need to change the `<INSERT>` variables to your own usernames and passwords.

### 5. Configure Apache

Generate a self-signed certificate by entering following commands on the host system. Enter following commands in a bash shell:

```bash
$ openssl genrsa -out pu001.key 2048

$ openssl req -new -key pu001.key -out pu001.csr

$ openssl x509 -req -days 365 -in pu001.csr -signkey pu001.key -out pu001.crt

```

Place these generated files in a folder **files** in `/ansible`.

Open the host file `pu001.yml` and enter following code:

```yaml
httpd_ssl_certificate_key_file: 'pu001.key'
httpd_ssl_certificate_file: 'pu001.crt'
```

Now go to the playbook `site.yml` and enter following code:

```yaml
  pre_tasks:
    - name: Copy web server private key
      copy:
        src: files/pu001.key
        dest: /etc/pki/tls/private/pu001.key
    - name: Copy web server certificate
      copy:
        src: files/pu001.crt
        dest: /etc/pki/tls/certs/pu001.crt
```
This code copies the generated files to the correct location because genrating the certificate should not be a part of our playbook.

---

## Test report

After running the test I noticed that the packages of wordpress end php-mysql were not installed. That was due to the fact that centos 8.2 did not support those packages yet at the time of making this assignment.

After using centos 7.6 i got following output:

```bash
[vagrant@pu001 /]$ sudo /vagrant/test/runbats.sh
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
Running test /vagrant/test/pu001/lamp.bats
 ✓ The necessary packages should be installed
 ✓ The Apache service should be running
 ✓ The Apache service should be started at boot
 ✓ The MariaDB service should be running
 ✓ The MariaDB service should be started at boot
 ✓ The SELinux status should be ‘enforcing’
 ✓ Web traffic should pass through the firewall
 ✓ Mariadb should have a database for Wordpress
 ✓ The MariaDB user should have write
 ✓ The website should be accessible through HTTP
 ✓ The website should be accessible through HTTPS
 ✓ The certificate should not be the default one
 ✓ The Wordpress install page should be visible under http://203.0.113.10/wordpress/
 ✓ MariaDB should not have a test database
 ✓ MariaDB should not have anonymous users

15 tests, 0 failures
```

---

## Resources

* [MariaDB User](https://docs.ansible.com/ansible/latest/collections/community/mysql/mysql_user_module.html)

* [Generating self-signed certificate](https://wiki.centos.org/HowTos/Https)

* README files from the installed roles