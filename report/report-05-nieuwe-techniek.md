# Enterprise Linux Lab Report

- Student name: Maximiliaan Muylaert
- Github repo: <https://github.com/HoGentTIN/elnx-2021-sme-MaximiliaanM>

---

## Test plan

---

## Procedure/Documentation

### Installing ansible on the host computer

To use Ansible Vault in this project we have to install Ansible on the host PC. If you have a Windows OS you have to use a workaround because Ansible is not supported on Windows.

1. Installing WSL

    Enable WSL feature with Powershell:

    ```powershell
    Enable-WindowsOptionalFeature -Online -FeatureName Microsoft-Windows-Subsystem-Linux
    ```

    To install a subsystem search te Microsoft Store for a WSL (e.g. I used **Ubuntu 18.04 LTS**)

2. Install Ansible

    To install Ansible enter the following commands:

    ```bash
    $ sudo apt update
    $ sudo apt install software-properties-common
    $ sudo apt-add-repository --yes --update ppa:ansible/ansible
    $ sudo apt install ansible
    ``` 

### Encrypting sensitive content with Ansible Vault

1.

---

## Test report


---

## Resources

* [Ansible Vault documentation](https://docs.ansible.com/ansible/latest/user_guide/vault.html#id1)
* [WSL installation](https://docs.microsoft.com/en-us/windows/wsl/install-win10)
* [Ansible Installation](https://docs.ansible.com/ansible/latest/installation_guide/intro_installation.html#installing-ansible-on-ubuntu)