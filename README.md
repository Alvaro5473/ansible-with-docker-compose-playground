# A playground to test Ansible with Docker Compose & Ubuntu

## Actions in the Host machine:

Launch containers
```bash
docker-compose up -d
```

![{F0D50863-64C1-45AD-A7E8-766146ADBB7E}](https://github.com/user-attachments/assets/ab1f282a-4e9a-49fa-a4ea-133fb5dd12fa)

Jump to the control node from host machine
```bash
docker-compose exec control bash
```

![{C5F0A81E-562F-48C6-8D13-F11FA3314B68}](https://github.com/user-attachments/assets/6ec92f76-96ac-4957-9c46-fa5c06a26900)

Cleanup
```bash
docker compose down
```

# Inside control node

Install ansible
```bash
sudo apt update
sudo apt-get install nano
sudo apt-get install software-properties-common
sudo apt-add-repository ppa:ansible/ansible
sudo apt-get install ansible
```

![{8A6F7433-5653-481B-9CA0-A68D24E66DC6}](https://github.com/user-attachments/assets/fc8e949a-5c95-4c54-a7ec-e5e88c6008e3)
![{E1F2FDB2-77E6-4D0F-A126-DFA31DFFD733}](https://github.com/user-attachments/assets/6b083e43-430e-42cb-8226-54331630e29f)
![{7915B03B-F4CE-439F-9BC6-11DCB5C53ECD}](https://github.com/user-attachments/assets/5d32f50f-2673-4354-a8a8-dd6345b71033)
![{87621F42-6920-4605-9BB0-AA8EF044ECC9}](https://github.com/user-attachments/assets/f1ecf185-71d0-4453-9cab-e9790861ba6d)
![{1E146EE5-FC94-4D4E-956C-139DA331D8AF}](https://github.com/user-attachments/assets/63f84a5d-388c-4e6f-acc2-cf3276d0d736)

Setup ansible
```bash
sudo nano /etc/ansible/hosts
```

Content of host the file:
```
[servers]
server1 ansible_host=node1
server2 ansible_host=node2
server3 ansible_host=node3

[all:vars]
ansible_connection=ssh
ansible_user=root
ansible_ssh_pass=password
ansible_python_interpreter=/usr/bin/python3
```

![{9BB1C066-88C1-4CC7-8922-E97A3F063F51}](https://github.com/user-attachments/assets/a60a129e-87fb-4f27-9760-71e2399af3df)

Test conectivity
```bash
ansible-inventory --list -y
ansible all -m ping -u root
ansible all -a "df -h" -u root
```

![{B4FA8984-41A0-47FA-A684-9874A8C099FA}](https://github.com/user-attachments/assets/99ea3165-4d3d-4ff8-b946-60c77a35bc0e)
![{E7310FE3-BCA7-4921-AC5A-194D4B5E93FF}](https://github.com/user-attachments/assets/4632deed-a055-47da-9811-4b11dbcbefa5)
![{DF6A3DDA-C022-4ED0-AC12-FAAF43157363}](https://github.com/user-attachments/assets/1ca4a389-26cc-40fd-9427-2dcc6046a644)

Connect with nodes (ssh password is "password" without quotes):
```bash
ssh root@node1
ssh root@node2
ssh root@node3
```

![{A2F771D3-39E2-4955-ACF2-1D856E6E6E1D}](https://github.com/user-attachments/assets/fc0067f1-fe10-4132-96c5-e98abe392727)
![{8111B17C-8C01-46AD-89D8-F05E40BE1282}](https://github.com/user-attachments/assets/fe0448e3-c226-4495-9ef7-5d88bffa9d59)
![{361E6EB5-B5F3-42D0-9A7F-526721123577}](https://github.com/user-attachments/assets/577fbe18-cddf-44fd-bb59-71b5014d8dba)

Shared folder between host machine and control node:
```bash
cd /shared
```

![{31A63ABD-22B8-4C1B-A8FB-5925574F2E3C}](https://github.com/user-attachments/assets/bd82ee95-2e76-4e21-a470-7af59eeb208a)

Expansion of the recipe to generate a process that takes advantage of open ports.

```bash
---
- name: Intro to Ansible Playbooks
  hosts: all

  tasks:
  - name: Copy file hosts with permissions
    ansible.builtin.copy:
      src: /etc/ansible/hosts
      dest: /tmp/hosts_ansible
      mode: '0644'
  
  - name: Add the user 'bob'
    ansible.builtin.user:
      name: bob
    become: yes
    become_method: sudo
  
  - name: Upgrade all apt packages
    apt:
      force_apt_get: yes
      upgrade: dist
    become: yes

  - name: Check for open ports
    ansible.builtin.command: netstat -tuln | grep LISTEN
    register: open_ports

  - name: Display open ports
    ansible.builtin.debug:
      var: open_ports.stdout_lines

  - name: Install nginx
    ansible.builtin.apt:
      name: nginx
      state: present
    become: yes

  - name: Ensure nginx is running and enabled
    ansible.builtin.systemd:
      name: nginx
      state: started
      enabled: yes
    become: yes

  - name: Configure nginx to listen on specific open ports
    ansible.builtin.template:
      src: nginx.conf.j2
      dest: /etc/nginx/sites-available/default
    notify:
      - Reload nginx

  - name: Open port 80 and 443 in UFW
    ansible.builtin.ufw:
      rule: allow
      name: 'Nginx Full'
    become: yes

  handlers:
  - name: Reload nginx
    ansible.builtin.systemd:
      name: nginx
      state: reloaded
    become: yes

```
