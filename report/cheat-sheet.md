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

## Ansible host config

| Task              | Command           | Arguments |
| :---              | :---              | :---      |
| Enable firewall services|`rhbase_firewall_allow_services`| https,  http|
| Enforce SELinux| `rhbase_selinux_state`| enforcing|
| Install the EPEL repository| `rhbase_repositories`| epel-release|
| Install packages| `rhbase_install_packages`| git, nano, vim-enhanced,...|
| Create Users| `rhbase_users`| name, comment, groups, password, ssh_key|
| Allow groups to SSH| `rhbase_ssh_allow_groups`| vagrant|
| Enable motd| `rhbase_dynamic_motd`| true, false|

## Git workflow

Simple workflow for a personal project without other contributors:

| Task                                         | Command                   |
| :---                                         | :---                      |
| Current project status                       | `git status`              |
| Select files to be committed                 | `git add FILE...`         |
| Commit changes to local repository           | `git commit -m 'MESSAGE'` |
| Push local changes to remote repository      | `git push`                |
| Pull changes from remote repository to local | `git pull`                |

## Checklist network configuration

1. Is the IP-adress correct? `ip a`
2. Is the router/default gateway correct? `ip r -n`
3. Is a DNS-server available? `cat /etc/resolv.conf`