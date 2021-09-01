# Enterprise Linux Lab Report

- Student name: Maximiliaan Muylaert
- Github repo: <https://github.com/HoGentTIN/elnx-2021-sme-MaximiliaanM>

---

## Test plan


---

## Procedure/Documentation

### 1. Add host

Add host `pr010` to the `vagrant-hosts.yml` file.

```yaml
  - name: pr010
    ip: 172.16.128.10
    netmask: 255.255.0.0
```
Create a new file in the `hosts_vars` folder named `pr010.yml`. Insert following code into the file.

```yaml
      # host_vars/pr010.yml
      # Variables visible to pr010
      ---
      #Firewall services
      rhbase_firewall_allow_services:
        - https
        - http

      #SELinux
      rhbase_selinux_state: 'enforcing'
```
### 2. Create docker-compose file & configure containers

Create a new file named `docker-compose.yml` in the folder `files` and insert following code to configure the necessary containers.

```yaml
version: '3.1'

services:

  wordpress:
    image: wordpress
    restart: always
    ports:
      - 81:80
    environment:
      WORDPRESS_DB_HOST: db
      WORDPRESS_DB_USER: exampleuser
      WORDPRESS_DB_PASSWORD: examplepass
      WORDPRESS_DB_NAME: exampledb
    volumes:
      - wordpress:/var/www/html

  db:
    image: mysql:5.7
    restart: always
    environment:
      MYSQL_DATABASE: exampledb
      MYSQL_USER: exampleuser
      MYSQL_PASSWORD: examplepass
      MYSQL_RANDOM_ROOT_PASSWORD: '1'
    volumes:
      - db:/var/lib/mysql

  portainer:
    image: portainer/portainer-ce
    restart: always
    ports:
      - 9000:9000
    command: -H unix:///var/run/docker.sock
    volumes:
      - /var/run/docker.sock:/var/run/docker.sock
      - portainer_data:/data

  haproxy:
    build: ./haproxy
    container_name: haproxy
    restart: always
    ports:
      - "80:80"

volumes:
  wordpress:
  db:
  portainer_data:
```
### 3. Install role and add tasks for host `pr010`

Open `site.yml` and add the new host `pr010` to the playbook.

```yaml
- hosts: pr010
  become: true
  roles:
    - bertvv.rh-base
  tasks:
    - name: Installing Docker & Docker Compose
      yum: 
        name: 
          - docker
          - docker-compose
        state: latest
    - name: Starting & enabling Docker
      systemd:
        name: docker
        state: started
        enabled: yes
    - name: Copy docker-compose file to VM
      copy:
        src: files/docker-compose.yml
        dest: /home/vagrant/docker-compose.yml
    - name: Copy dockerfile for HAProxy to VM
      copy:
        src: files/Dockerfile
        dest: /home/vagrant/haproxy/Dockerfile
    - name: Copy HAProxy config file to VM
      copy:
        src: files/haproxy.cfg
        dest: /home/vagrant/haproxy/haproxy.cfg
    - name: Start containers with docker-compose
      shell:
        chdir: /home/vagrant/
        cmd: docker-compose up -d
```
### 4. Add necessary config files for HAProxy

Create a Dockerfile named `Dockerfile` to the folder `files` with following code.

```dockerfile
FROM haproxy:latest
COPY haproxy.cfg /usr/local/etc/haproxy/haproxy.cfg
```

Create the HAProxy config file named `haproxy.cfg` and place it also in the folder `files`

```
global
  stats socket /var/run/api.sock user haproxy group haproxy mode 660 level admin expose-fd listeners
  log stdout format raw local0 info

defaults
  mode http
  timeout client 10s
  timeout connect 5s
  timeout server 10s
  timeout http-request 10s
  log global

frontend stats
  bind *:8404
  stats enable
  stats uri /
  stats refresh 10s

frontend myfrontend
  bind :80
  default_backend webservers

backend webservers
  server s1 81:8080 check
```
    
---

## Test report


---

## Resources

* [Ansible Vault documentation](https://docs.ansible.com/ansible/latest/user_guide/vault.html#id1)
* [WSL installation](https://docs.microsoft.com/en-us/windows/wsl/install-win10)
* [Ansible Installation](https://docs.ansible.com/ansible/latest/installation_guide/intro_installation.html#installing-ansible-on-ubuntu)