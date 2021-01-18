# Enterprise Linux Lab Report

- Student name: Maximiliaan Muylaert
- Github repo: <https://github.com/HoGentTIN/elnx-2021-sme-MaximiliaanM>

---

## Test plan

1. Start all servers with: `vagrant up`

2. Create a new VirtualBox VM manually, give it two host-only network interfaces, both attached to the VirtualBox host-only network with IP 172.16.0.0/16.

3. Boot the VM with a LiveCD ISO (e.g. Fedora, but Ubuntu, Kali, etc. should also be fine).

4. Things to check (follow the guidelines for bottom-up troubleshooting!):

- **Network access layer**
    - Is the workstation connected to the correct VirtualBox Host-only network?
    - Is any NAT or bridged interface disabled (or cable disconnected)?
- **Internet layer**
    - Did the VM receive correct IP settings from the DHCP server?
        - IP address in the correct range for either guest with dynamic IP or reserved IP?
        - Correct subnet mask?
        - DNS server?
        - Default gateway?
    - LAN connectivity: can you ping
        - other hosts in the same network?
        - the gateway? All its IP addresses?
        - a host in the DMZ?
        - the default gateway of the router?
    - Is the DNS server responsive?
        - does it resolve www.avalon.lan?
            - does it resolve an external name? (e.g. www.google.com, icanhazip.com)
        - does it resolve reverse lookups for avalon.lan? (e.g. 203.0.113.10, 172.16.128.1)
- **Transport layer**
    - Not applicable, as no services run on the workstation
- **Application layer**: Are network services available?
    - Is <http://www.avalon.lan/wordpress/> visible?
    - Is an external website, e.g. <http://icanhazip.com/>, visible?
    - is the fileserver available? e.g. smb://files/public or `ftp files.avalon.lan`.

---

## Procedure/Documentation

### Configure the router

To configure the settings of the VyOS router open `scripts\router-config.sh` and enter following code:

1. Configure the basic settings

    ```bash
    set system host-name 'router'
    set service ssh port '22'
    ```

2. Configure the network interfaces

    ```bash
    set interfaces ethernet eth0 address dhcp
    set interfaces ethernet eth0 description WAN

    set interfaces ethernet eth1 address 203.0.113.254/24
    set interfaces ethernet eth1 description DMZ

    set interfaces ethernet eth2 address 172.16.255.254/16
    set interfaces ethernet eth2 description internal
    ```

3. Configure Network Address Translation (NAT)

    ```bash
    set nat source rule 200 outbound-interface 'eth0'
    set nat source rule 200 source address '172.16.0.0/24'
    set nat source rule 200 translation address 'masquerade'

    set nat source rule 300 outbound-interface 'eth1'
    set nat source rule 300 source address '172.16.0.0/24'
    set nat source rule 300 translation address 'masquerade'
    ```

4. Configure Network Time Protocol (NTP)

    ```bash
    delete system ntp server 0.pool.ntp.org
    delete system ntp server 1.pool.ntp.org
    delete system ntp server 2.pool.ntp.org
    set system ntp server 'be.pool.ntp.org'
    set system time-zone Europe/Brussels
    ```

5. Configure forwarding DNS server

    ```bash
    set service dns forwarding domain avalon.lan server 172.16.128.1
    set service dns forwarding name-server system
    set service dns forwarding listen-on 'eth1'
    set service dns forwarding listen-on 'eth2'
    ```

### Configure DHCP

1. Add `pr003` as a vagrant host:

    To set as host open the `vagrant-hosts.yml` file and enter following code:

    ```yaml
    - name: pr003
    ip: 172.16.128.3
    netmask: 255.255.0.0
    ```

2. Configure playbook & installing roles:

    Open the playbook `site.yml` and add the hosts **pr003**. install the roles `bertvv.rh-base` and `bertvv.dhcp`.

    ```yaml
    - hosts: pr003
      become: true
      roles:
        - bertvv.rh-base
        - bertvv.dhcp
    ```

    After adding them to the `ansible\site.yml` file, we're going to install them. Open a bash shell in the main directory and run following script: `./scripts/role-deps.sh`.


3. Configure host `pr003`

    Make a new file called `pr003.yml` in `host_vars` and enter following code to configure DHCP: 

    ```yaml
    #Firewall allow DHCP
    rhbase_firewall_allow_services:
    - dhcp

    #DHCP
    dhcp_global_domain_name: avalon.lan
    dhcp_global_domain_name_servers: 172.16.255.254
    dhcp_global_subnet_mask: 255.255.0.0
    dhcp_global_routers: 172.16.255.254
    dhcp_global_default_lease_time: 43200
    dhcp_global_max_lease_time: 43200
    dhcp_subnets:
    - ip: 172.16.0.0
        netmask: 255.255.0.0
        pools:
        - default_lease_time: 14400
            max_lease_time: 14400
            range_begin: 172.16.0.2
            range_end: 172.16.127.254
            allow: unknown-clients
        - range_begin: 172.16.192.1
            range_end: 172.16.255.254
            deny: unknown-clients
    dhcp_hosts:
    - name: kali
        mac: '08:00:27:38:a0:ec'
        ip: 172.16.192.2
    ```

---

## Test report

1. Start all servers with: `vagrant up`

2. Create a new VirtualBox VM manually, give it two host-only network interfaces, both attached to the VirtualBox host-only network with IP 172.16.0.0/16.

3. Boot the VM with a LiveCD ISO (e.g. Fedora, but Ubuntu, Kali, etc. should also be fine).

4. Things to check (follow the guidelines for bottom-up troubleshooting!):

- **Network access layer**
    - Is the workstation connected to the correct VirtualBox Host-only network?
    - Is any NAT or bridged interface disabled (or cable disconnected)?
- **Internet layer**
    - Did the VM receive correct IP settings from the DHCP server?
        - IP address in the correct range for either guest with dynamic IP or reserved IP?
        - Correct subnet mask?

            ![ip a](img/ip-a.jpg)
        - DNS server?

            ![cat /etc/resolv.conf](img/etc-resolveconf.jpg)
        - Default gateway?

            ![ip r](img/ip-r.jpg)
        
    - LAN connectivity: can you ping
        - other hosts in the same network?
        
            ![ping 172.16.128.1](img/ping-ns1.jpg)
        - the gateway? All its IP addresses?

            ![ping 172.16.255.254](img/ping-gateway.jpg)
        - a host in the DMZ?

            ![ping 203.0.113.10](img/ping-web.jpg)
        - the default gateway of the router?

            ![ping 203.0.113.254](img/ping-gateway-router.jpg)
    - Is the DNS server responsive?
        - does it resolve www.avalon.lan?

            ![dig www.avalon.lan](img/dig-avalon.jpg)
        - does it resolve an external name? (e.g. www.google.com, icanhazip.com)

            ![dig icanhazip.com](img/dig-icanhazip.jpg)
        - does it resolve reverse lookups for avalon.lan? (e.g. 203.0.113.10, 172.16.128.1)

            ![dig -x 172.16.128.1](img/dig-reverse.jpg)
- **Transport layer**
    - Not applicable, as no services run on the workstation
- **Application layer**: Are network services available?
    - Is <http://www.avalon.lan/wordpress/> visible?

        ![www.avalan.lan/wordpress](img/avalon-wordpress.jpg)
    - Is an external website, e.g. <http://icanhazip.com/>, visible?

        ![Icanhazip.com](img/icanhazip.jpg)
    - is the fileserver available? e.g. smb://files/public or `ftp files.avalon.lan`.

        ![ftp](img/ftp.jpg)
        ![smb](img/smb.jpg)

---

## Resources

* [VyOS Documentation](https://docs.vyos.io/en/latest/index.html)
* [NTP pool Belgium](https://www.pool.ntp.org/zone/be)
