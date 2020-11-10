# Enterprise Linux Lab Report

- Student name: Maximiliaan Muylaert
- Github repo: <https://github.com/HoGentTIN/elnx-2021-sme-MaximiliaanM>

---

## Test plan



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
    set service dns forwarding name-server 10.0.2.3
    set service dns forwarding listen-on 'eth1'
    set service dns forwarding listen-on 'eth2'
    ```

### Configure DHCP
---

## Test report



---

## Resources

* [VyOS Documentation](https://docs.vyos.io/en/latest/index.html)
