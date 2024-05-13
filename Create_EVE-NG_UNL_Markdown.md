### ネットワークトポロジー

                 192.168.1.0/24         
          .10                          .11
   【Linux】-------【Cloud0】-----------【R1】





### R1 (Cisco IOL) の設定
```plaintext
hostname R1
ip domain name flex-one.com
crypto key generate rsa modulus 2048

username cisco password 0 cisco

interface Ethernet0/0
 ip address 192.168.1.11 255.255.255.0
 no shut

line vty 0 4
 login local
 transport input all

end
wr
```

### Linux サーバーの設定 (/etc/network/interfaces)
``` #iface ens3 inet dhcp
iface ens3 inet static
 address 192.168.1.10
 network 192.168.1.0
 netmask 255.255.255.0
 broadcast 192.168.1.255
 gateway 192.168.1.1
 dns-nameserver 8.8.8.8
```


```apt-get install -y software-properties-common
apt-add-repository ppa:ansible/ansible
apt-get update
apt-get install -y ansible
```


### インベントリファイル (hosts.yml)
```all:
  hosts:
    R1:
      ansible_host: 192.168.1.11
      ansible_connection: network_cli
      ansible_network_os: ios
      ansible_user: cisco
      ansible_password: cisco
```

### プレイブック (show_version.yml)
```- hosts: R1
  gather_facts: no

  tasks:
    - name: send show version
      ios_command:
        commands: "show version"
      register: ios_results

    - name: display show version
      debug:
        var: ios_results
```

### SSH接続テストとフィンガープリント
```root@ubuntu:~# ssh cisco@192.168.1.11
The authenticity of host '192.168.1.11 (192.168.1.11)' can't be established.
RSA key fingerprint is SHA256:iSsO89zt2eoJhHcElm2kYRXf9IUFsSM9ezJhOdcsj2k.
Are you sure you want to continue connecting (yes/no)? yes
Warning: Permanently added '192.168.1.11' (RSA) to the list of known hosts.
Password:

R1>exit
```

### Ansible プレイブックの実行
```root@ubuntu:~# ansible-playbook -i hosts.yml show_version.yml

PLAY [R1] **********************************************************************

TASK [send show version] *******************************************************
ok: [R1]

TASK [display show version] ****************************************************
ok: [R1] => {
    "ios_results": {
        "ansible_facts": {
            "discovered_interpreter_python": "/usr/bin/python"
        },
        "changed": false,
        "deprecations": [
            {
                "msg": "Distribution Ubuntu 16.04 on host R1 should use /usr/bin/python3, but is using /usr/bin/python for backward compatibility with prior Ansible releases. A future Ansible release will default to using the discovered platform python for this host."
            }
        ],
        "failed": false,
        "stdout": [
            "Cisco IOS Software, Linux Software (I86BI_LINUX-ADVENTERPRISEK9-M), Version 15.5(2)T, DEVELOPMENT TEST SOFTWARE",
            ...
        ],
        "stdout_lines": [
            [
                "Cisco IOS Software, Linux Software (I86BI_LINUX-ADVENTERPRISEK9-M), Version 15.5(2)T, DEVELOPMENT TEST SOFTWARE",
                ...
            ]
        ]
    }
}

PLAY RECAP *********************************************************************
R1                         : ok=2    changed=0    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0
```
