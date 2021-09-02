# Enterprise Linux Lab Report

- Student name: Maximiliaan Muylaert
- Github repo: <https://github.com/MaximiliaanM/mc_overviewer>
- Ansible Galaxy: <https://galaxy.ansible.com/maximiliaanm/mc_overviewer>

---

## Documentation

### 1. Creating a role

Using a WSL on windows, enter following command in the terminal:

```bash
    $ ansible-galaxy init mc_overviewer
```
the directory will look like this:
```bash
    .travis.yml
    README.md
    defaults/
        main.yml
    files/
    handlers/
        main.yml
    meta/
        main.yml
    tasks/
        main.yml
    templates/
    tests/
        inventory
        test.yml
    vars/
        main.yml
```

### 2. Make role functional

Open the folder `meta` and add following platforms to the `main.yaml` file.

```yaml
platforms:
  - name: Fedora
    versions:
    - all
  - name: CentOS
    versions:
    - all
  - name: Debian
    versions:
    - all
  - name: Ubuntu
    versions:
    - all
```

Then open the `tasks` folder and add following code:

```yaml
# tasks file for mc_overviewer
- name: Installing Overviewer for Fedora / CentOS
  block:
    - name: Enable EPEL to get a release of Python 3
      yum_repository:
        name: epel-repo
        baseurl: https://dl.fedoraproject.org/pub/epel/epel-release-latest-7.noarch.rpm
    - name: Add the Overviewer repository to yum
      get_url:
        url: https://overviewer.org/rpms/overviewer.repo
        dest: /etc/yum.repos.d/overviewer.repo
    - name: Install the Overviewer
      yum:
        name: Minecraft-Overviewer
        state: present
        update_cache: yes
    - name: Ensure overviewer conf is set
      template:
        src: conf.j2
        dest: /tmp/overviewer.conf
    - name: Execute overviewer
      shell: overviewer.py --config=/tmp/overviewer.conf
  when: ansible_facts['os_family'] == "RedHat"  

- name: Installing Overviewer for Debian / Ubuntu
  block:
    - name: Ensure apt-transport-https is installed
      apt:
        name: apt-transport-https
        state: present
        update_cache: yes
    - name: Add the Overviewer repository to the apt sources list
      lineinfile:
        path: /etc/apt/sources.list
        state: present
        insertafter: EOF
        line: deb https://overviewer.org/debian ./
    - name: Install key for signed repository
      apt_key:
        url: https://overviewer.org/debian/overviewer.gpg.asc
        state: present
    - name: Install the Overviewer
      apt: 
        name: minecraft-overviewer
        state: present
        update_cache: yes
    - name: Ensure overviewer conf is set
      template:
        src: conf.j2
        dest: /tmp/overviewer.conf
    - name: Execute overviewer
      shell: overviewer.py --config=/tmp/overviewer.conf
  when: ansible_facts['os_family'] == "Debian"
```
Lastly make a new file named `conf.j2` in the folder templates and insert following code:

```j2
{{ % for world in worlds % }}
worlds["{{ world.name }}"] = "{{ world.path }}"

renders["{{ world.render }}"] = {
    "world": "{{ world.name }}",
    "title": "{{ world.title }}",
    "dimension": "{{ world.dimension }}",
}

{{ % endfor % }}
outputdir = "{{ outputdir }}"
```

### 3. Clean role and uploud to Ansible Galaxy

Now delete every folder that we did not use to make the role cleaner.

To upload the role to Ansible Galaxy we need to place it on Github as a repository. First make a new repository on github and copy the repo link. Then open a terminal in the role's location and follow these steps:

```bash
git init

git add *

git commit -m "Initial commit"

git remote add origin https://github.com/MaximiliaanM/mc_overviewer.git

git push origin master
```
Now go to Ansible Galaxy, login with github credentials and got to the My Content tab and click the `Add Content` button. Choose your github repo and you're finished.

---

## Resources

* [Creating Roles](https://galaxy.ansible.com/docs/contributing/creating_role.html)
* [Create and push roles to Ansible Galaxy](https://www.opcito.com/blogs/how-to-write-ansible-roles-and-publish-them-on-ansible-galaxy)
* [yum](https://docs.ansible.com/ansible/latest/collections/ansible/builtin/yum_module.html)
* [apt](https://docs.ansible.com/ansible/latest/collections/ansible/builtin/apt_module.html)
* [apt_key](https://docs.ansible.com/ansible/latest/collections/ansible/builtin/apt_key_module.html)
* [template](https://docs.ansible.com/ansible/latest/collections/ansible/builtin/template_module.html)
* [get_url](https://docs.ansible.com/ansible/latest/collections/ansible/builtin/get_url_module.html)
* [yum_repository](https://docs.ansible.com/ansible/latest/collections/ansible/builtin/yum_repository_module.html)
* [shell](https://docs.ansible.com/ansible/latest/collections/ansible/builtin/shell_module.html)
* [lineinfile](https://docs.ansible.com/ansible/latest/collections/ansible/builtin/lineinfile_module.html)