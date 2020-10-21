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

```

---

## Procedure/Documentation

### 1. Adding `pr001` and `pr002` as a vagrant host

To set these as hosts open the `vagrant-hosts.yml` file and enter following code:

```yaml
- name: pr001
  ip: 172.16.128.1
- name: pr002
  ip: 172.16.128.2
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

Make 2 new files called `pr001.yml` and `pr002.yml` in `ansible\host_vars`. To configure these files follow next steps(these steps need to be repeated for `pr002`):

1. **Allow dns on the firewall by entering following code:**

    ```yaml
    rhbase_firawall_allow_services:
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
    - name: avlon.lan
        type: primary #Primary DNS-Server
        create_forward_zones: true
        create_reverse_zones: true
        primaries: 
        - 172.16.128.1
        name_servers:
        - pr001.avlon.lan.
        - pr002.avlon.lan.
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

```yaml
mail_servers:
  - name: pu002
    preference: 10
```

---

## Test report

The test report is a transcript of the execution of the test plan, with the actual results. Significant problems you encountered should also be mentioned here, as well as any solutions you found. The test report should clearly prove that you have met the requirements.

---

## Resources

* [Dig command](https://www.a2hosting.com/kb/getting-started-guide/internet-and-networking/troubleshooting-dns-with-dig-and-nslookup)
* `ansible\roles\bertvv.bind\README.md`
