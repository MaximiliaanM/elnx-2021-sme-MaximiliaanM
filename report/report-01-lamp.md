# Enterprise Linux Lab Report

- Student name: Maximiliaan Muylaert
- Github repo: <https://github.com/HoGentTIN/elnx-2021-sme-MaximiliaanM>

---

## Test plan

How are you going to verify that the requirements are met? The test plan is a detailed checklist of actions to take, including the expected result for each action, in order to prove your system meets the requirements. Part of this is running the automated tests, but it is not always possible to validate *all* requirements throught these tests.

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

The test report is a transcript of the execution of the test plan, with the actual results. Significant problems you encountered should also be mentioned here, as well as any solutions you found. The test report should clearly prove that you have met the requirements.

---

## Resources

* [MariaDB User](https://docs.ansible.com/ansible/latest/collections/community/mysql/mysql_user_module.html)

* [Generating self-signed certificate](https://wiki.centos.org/HowTos/Https)

* README files from the installed roles