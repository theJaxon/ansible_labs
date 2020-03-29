# ansible_labs
Part of the continuous delivery module.

### Lab 1:
#### Screens:
<details>
<summary>EC2 after installing Apache web server</summary>
<p>



</p>
</details>  

#### Steps:
1. EC2 instance launched allowing HTTP traffic on port 80
2. SSH connections were allowed on port 22
3. `~/.ssh/id_rsa.pub` was copied to the instance `~/.ssh/authorized_keys`
4. `ansible-playbook firstplaybook.yaml`

```yaml
---
- hosts: EC
  remote_user: ec2-user
  become: true
  tasks:
  - name: install apache
    package:
      name: httpd
      state: present
  - name: start apache web server
    service:
      name: httpd
      state: started
  - name: write to the web page
    copy:
      src: index.html
      dest: /var/www/html/
```

### Lab 2:
#### Screens:
<details>
<summary>go-cd server running and nexus failing to start</summary>
<p>


</p>
</details>  

#### Steps:
1. Two roles were added with the following commands:
   * $ `ansible-galaxy init gocd`
   * $ `ansible-galaxy init nexus`

2- The following yaml file located at `gocd/tasks/main.yml` was used:
```yaml
---
# tasks file for gocd
- name: Download the .repo file for gocd and save it to /etc/yum/repos.d/
  get_url: 
    url: https://download.gocd.org/gocd.repo
    dest: /etc/yum.repos.d/gocd.repo

- name: Install gocd server 
  package:
    name: go-server
    state: present

- name: start gocd server 
  service:
    name: go-server 
    state: started

- name: install gocd agent
  package:
    name: go-agent 
    state: present 

- name: start gocd agent 
  service: 
    name: go-agent
    state: started 
```

3- The file located at `nexus/tasks/main.yml` was used for the second task:
```yaml
---
# tasks file for nexus
- name: install OpenJDK 1.8 
  package:
    name: java-1.8.0-openjdk.x86_64
    state: present

- name: creating app directory
  file:
    path: /app
    state: directory 

- name: download the latest nexus 
  get_url:
    url: https://download.sonatype.com/nexus/3/latest-unix.tar.gz
    dest: /app/

- name: extract downloaded archive to the current directory 
  unarchive: 
    src: /app/nexus-3.22.0-02-unix.tar.gz
    dest: /app/
    remote_src: yes

- name: remove the downloaded archive file 
  shell: >
    rm -rf /app/*.gz

- name: rename the extracted file to 'nexus'
  shell: >
    mv /app/nexus-3* /app/nexus

- name: add nexus user 
  user: 
    name: nexus 
    comment: Nexus user 

- name: change ownership of /app/nexus and /app/sonatype-network to the nexus user 
  file:
    path: "{{ item }}"
    owner: nexus 
  loop:
    - /app/nexus 
    - /app/sonatype-work 

- name: copy nexus.service file to /etc/systemd/system/
  copy:
    src: nexus.service 
    dest: /etc/systemd/system/

- name: start the nexus service 
  service: 
    name: nexus 
    state: started 

- name: allow nexus on boot
  systemd:
    name: nexus 
    enabled: yes
```

4- `main.yml` was used to execute the scripts:
```yaml
---
- hosts: all
  remote_user: ec2-user 
  become: true
  roles:
    - gocd
    - nexus
```

The command used: 
$ `ansible-playbook -i inventory main.yaml`