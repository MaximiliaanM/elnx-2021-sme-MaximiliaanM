# Enterprise Linux Lab Report

- Student name: Maximiliaan Muylaert
- Github repo: <https://github.com/HoGentTIN/elnx-2021-sme-MaximiliaanM>

---

## Test plan
### pu001
1. Start the VM with `vagrant up pu001` in a bash shell.
2. Log in into the VM with `vagrant ssh pu001`.
3. Run the test file with `sudo /vagrant/test/runbats.sh`.
### pr011
1. Start the vm with `vagrant up pr011`.
2. SSH into `pr011` with `vagrant ssh pr011`.
3. Excute the test with `sudo /vagrant/test/runbats.sh`
---

## Procedure/Documentation

### Installing ansible on the host computer

To use Ansible Vault in this project we have to install Ansible on the host PC. If you have a Windows OS you have to use a workaround because Ansible is not supported on Windows.

1. Installing WSL

    Enable WSL feature with Powershell:

    ```powershell
    Enable-WindowsOptionalFeature -Online -FeatureName Microsoft-Windows-Subsystem-Linux
    ```

    To install a subsystem search the Microsoft Store for a WSL (e.g. I used **Ubuntu 18.04 LTS**)

2. Install Ansible

    To install Ansible enter the following commands:

    ```bash
    $ sudo apt update
    $ sudo apt install software-properties-common
    $ sudo apt-add-repository --yes --update ppa:ansible/ansible
    $ sudo apt install ansible
    ``` 

### Encrypting sensitive content with Ansible Vault

1. Make a file wich contains our Ansible Vault password (e.g.: `vault.txt` contains ILoveLinux2 as Ansible Vault password)

2. Encrypt all password used in `pu001` & `pr011`
    
    Use following command in the WSL to encrypt our passwords.
    ```
    ansible-vault encrypt_string --vault-password-file vault.txt '<PASSWORD_TO_ENCRYPT>' --name '<VAULT_ID>'
    ```

    1. `pu001`:

    Mariadb_user
    ```
    maximiliaan@PC-Maximiliaan:~$ ansible-vault encrypt_string --vault-password-file vault.txt 'ILoveLinux2' --name 'password'
    password: !vault |
          $ANSIBLE_VAULT;1.1;AES256
          33306238393339656664353436376166376531353066316463623834663932346236393037333131
          6231656364333164353232336433656164313166373662660a633638616130373735353238373939
          33343661643536663065326563353233636366393466383463383266306230343138313064323433
          3333353039333433340a333938663231633434393439396533333665656330613666663739373534
          3665
    ```
    mariadb_root_password
    ```
    maximiliaan@PC-Maximiliaan:~$ ansible-vault encrypt_string --vault-password-file vault.txt 'ILoveLinux2' --name 'mariadb_root_password'
    mariadb_root_password: !vault |
          $ANSIBLE_VAULT;1.1;AES256
          33633130663534646561643133396366653937663034366638653961663539623834613932306464
          3434636530616135666530303138393465363933326138660a656661613130376332393637303635
          65356337316332353563303832376466363133336635626262366161346534643266656331353537
          6338663363623237330a323338623037656632666137383666313938646364346132346631313661
          6133
    ```
    wordpress_password
    ```
    maximiliaan@PC-Maximiliaan:~$ ansible-vault encrypt_string --vault-password-file vault.txt 'ILoveLinux2' --name 'wordpress_password'
    wordpress_password: !vault |
          $ANSIBLE_VAULT;1.1;AES256
          65646630666237393232373830333936366533323633636630613866313236656633386231333663
          3632353438646232383438643532373264396331613536650a613566336333333330623365363236
          62373239316236613036316138313161626231316236333666656636306633623632356130356164
          3630333530383737330a343338313837636364336561343464313837303364336461333132313039
          3031
    ```

    2. `pr011`:

    stevenh
    ```
    maximiliaan@PC-Maximiliaan:~$ ansible-vault encrypt_string --vault-password-file vault.txt 'stevenh' --name 'password'
    password: !vault |
          $ANSIBLE_VAULT;1.1;AES256
          32656536366335653330386561633835656231386162623630663565633161316339623965316639
          3436366161316337386431613335306236653465323932340a633565363161656533353865313964
          62613163656463633531626631663732396235393831343663383863396132643634643561663363
          3739393264666631310a616337363561626437613438366163373834383135613165336464356261
          6561
    ```
    stevenv
    ```
    maximiliaan@PC-Maximiliaan:~$ ansible-vault encrypt_string --vault-password-file vault.txt 'stevenv' --name 'password'
    password: !vault |
          $ANSIBLE_VAULT;1.1;AES256
          34346330613961376235666337396239323731306534646262316438343366333531386431363865
          3331323862376139393165613265316363366461306264610a356539393137623863363165636238
          34633836393164303362316264613266623862383364363830323733366661623234666665653137
          3866643236383961620a343762346663383539643161333966636331346431303934613036343238
          6463
    ```
    leend
    ```
    maximiliaan@PC-Maximiliaan:~$ ansible-vault encrypt_string --vault-password-file vault.txt 'leend' --name 'password'
    password: !vault |
          $ANSIBLE_VAULT;1.1;AES256
          64396431363534306431353962633137393231333638656566336463373731323530336133383866
          6261373433316237666166643861663662643165663433640a646435613937376464386565353635
          34383031646531383333306663646637343264653933373237343261313932373738653664656630
          6335333466306333370a653136383861393666613664363734323533316635383739613465623466
          6236
    ```
    svena
    ```
    maximiliaan@PC-Maximiliaan:~$ ansible-vault encrypt_string --vault-password-file vault.txt 'svena' --name 'password'
    password: !vault |
          $ANSIBLE_VAULT;1.1;AES256
          62303862613963323266336466646634646163623565656266356133343461333566303133636461
          6233613365663433373935613038373862333861663563320a643438653637383762383062616161
          38353139383264353533343236323434383461643531303637373735306132333531356462616265
          6434616362383830630a323665643963383436346163396363373537393762616134346335643433
          3433
    ```
    nehirb
    ```
    maximiliaan@PC-Maximiliaan:~$ ansible-vault encrypt_string --vault-password-file vault.txt 'nehirb' --name 'password'
    password: !vault |
          $ANSIBLE_VAULT;1.1;AES256
          35323461616535636633343765643232633035393736306463663130646662313366313435313432
          3030353130303934363636643631396530356238306431640a323162396365383662356135646430
          32376566393333356566313433383866623638303337363637666232623065366533343335653764
          3564373065636337390a626332623831613061656164316434646331663938396633343932326131
          3062
    ```
    alexanderd
    ```
    maximiliaan@PC-Maximiliaan:~$ ansible-vault encrypt_string --vault-password-file vault.txt 'alexanderd' --name 'password'
    password: !vault |
          $ANSIBLE_VAULT;1.1;AES256
          61383065313934656162623865313733396463313832363662613130616631393530346464336132
          6133373264383633396464356538663866383235366165630a653631383836666231356165373863
          30373363373865653134666137633138333738396561386439653430333336353532303539363632
          3935303238656462340a663264653565313631376637623035303464313838666132393564363033
          3031
    ```
    krisv
    ```
    maximiliaan@PC-Maximiliaan:~$ ansible-vault encrypt_string --vault-password-file vault.txt 'krisv' --name 'password'
    password: !vault |
          $ANSIBLE_VAULT;1.1;AES256
          35663131663965646530383566383530383735346139306132633466363866383130333039363932
          3766616235613662623764313939356633666563343065610a346639633434343932633563373262
          39393534366438633564666130373165663533383336656636633265323432636139663362396436
          6165326662646434630a313332303335616334656362656435626661343030613531653834636638
          6439
    ```
    benoitp
    ```
    maximiliaan@PC-Maximiliaan:~$ ansible-vault encrypt_string --vault-password-file vault.txt 'benoitp' --name 'password'
    password: !vault |
          $ANSIBLE_VAULT;1.1;AES256
          36323165386462643033666165626461386637623830613032306432626661366536663533343231
          6132376264343433383135383265313966343536383030390a326536616332303236616464636238
          66383866653537343935663761353935646231326564356539653731666332653830653235366139
          3035356266343732630a356633346138393464396664626637393834653763633032373363306266
          3133
    ```
    anc
    ```
    maximiliaan@PC-Maximiliaan:~$ ansible-vault encrypt_string --vault-password-file vault.txt 'anc' --name 'password'
    password: !vault |
          $ANSIBLE_VAULT;1.1;AES256
          62646233633836653766643561363636313761376562633865636265313963636562386665646231
          3734663038376438333539626635633635666262393234350a663530666663303266336431363635
          63663735623464353333333164333663653135313333666265393336383737663238333662346339
          3861386336653531630a623535656563386634306662633864623831346164363332613536626663
          3466
    ```
    elenaa
    ```
    maximiliaan@PC-Maximiliaan:~$ ansible-vault encrypt_string --vault-password-file vault.txt 'elenaa' --name 'password'
    password: !vault |
          $ANSIBLE_VAULT;1.1;AES256
          31663731323937393239373030633162323839623766376430663537613538313833656365333932
          3364633934623561333430633433636366623737353965370a633833346637366563643836333666
          66336364336435623337396233636433343338353535396437393032663364343931323463363765
          6234383438636633350a306338353761353866646435623735383338636261386137386338636464
          6538
    ```
    evyt
    ```
    maximiliaan@PC-Maximiliaan:~$ ansible-vault encrypt_string --vault-password-file vault.txt 'evyt' --name 'password'
    password: !vault |
          $ANSIBLE_VAULT;1.1;AES256
          30643030616566316362313339633737633435306231396139663735353334393330633964653562
          3639356661633430333036366235643830393436323066660a666136313539616139353062616562
          61363930343065333262636135393339346161646565376162313935643662323934656161376363
          3664393734343762300a313537303430613539333232363935343366393164633433656262616464
          3361
    ```
    christophev
    ```
    maximiliaan@PC-Maximiliaan:~$ ansible-vault encrypt_string --vault-password-file vault.txt 'christophev' --name 'password'
    password: !vault |
          $ANSIBLE_VAULT;1.1;AES256
          37346264363865393730323131366238646334656539353435373964626439626235333162623965
          3538623134316133373837356338356530356664323435660a643466333264646434616462316139
          34653963336139656330613930306431316263353961343932303537373935653638366365303931
          3239636631643338350a383136656438356362356366626163373265303733313230306638363138
          6536
    ```
    stefaanv
    ```
    maximiliaan@PC-Maximiliaan:~$ ansible-vault encrypt_string --vault-password-file vault.txt 'stefaanv' --name 'password'
    password: !vault |
          $ANSIBLE_VAULT;1.1;AES256
          62336438366438646437393037613364663533393238313130303439623465646133323134383033
          3234356533643264656263303236653938373366623233370a643839643137623137313933373762
          38393630306161306265393762623838313963636164363536393632396332643036396434376634
          3833376337653165300a653463373235313265323663346131653938663463303266313763306237
          6665
    ```
    maximiliaan
    ```
    maximiliaan@PC-Maximiliaan:~$ ansible-vault encrypt_string --vault-password-file vault.txt 'maximiliaan' --name 'password'
    password: !vault |
          $ANSIBLE_VAULT;1.1;AES256
          38346631656632396632306563353135313835633233633034653461393031363665313534383836
          3134666134656661383566396537373561303235663934650a626336353434383236323238326634
          35383432343139323063633962663136363538666636373962623039613235393534663432343039
          3864396333386665320a663333383966383430646662316236663062666664346664353966313865
          6566
    ```

3. Edit `Vagrantfile` to allow decryption for running the playbook

    1. Add following line after `ansible.become = true` at line 113
    ```
    ansible.vault_password_file = "/tmp/vault_pass"
    ```
    This gives the path of our Ansible Vault password that is used to decrypt the passwords

    2. Add following line after `hosts.each do |host|` at line 156
    ```
    config.vm.provision :shell, inline: "echo 'ILoveLinux2' > /tmp/vault_pass"
    ```
    This line is added because of problems I had with the permissions, it kept giving the same error that it should not be executable so I removed the execute permission but still got the same error. This line 'hardcodes' the password so that i can still be decrypted.

    ```
    [WARNING]: Error in vault password file loading (default): Problem running
    vault password script /vagrant/ansible/files/vault ([Errno 8] Exec format
    error). If this is not a script, remove the executable bit from the file.
    ERROR! Problem running vault password script /vagrant/ansible/files/vault ([Errno 8] Exec format error). If this is not a script, remove the executable bit from the file.
    Ansible failed to complete successfully. Any error output should be
    visible above. Please fix these errors and try again.
    ```
    
---

## Test report



3. Test results of `pu001`

```bash
[vagrant@pu001 ~]$ sudo /vagrant/test/runbats.sh
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

4. Test results of `pr011`

```bash
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

* [Ansible Vault documentation](https://docs.ansible.com/ansible/latest/user_guide/vault.html#id1)
* [WSL installation](https://docs.microsoft.com/en-us/windows/wsl/install-win10)
* [Ansible Installation](https://docs.ansible.com/ansible/latest/installation_guide/intro_installation.html#installing-ansible-on-ubuntu)