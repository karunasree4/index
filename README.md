Below are the details of the test. The test is divided into 6 steps, beginning with setting up the infrastructure to
installing software and configuring the servers. Our preferred infrastructure automation tools are Terraform and
Ansible, but it’s not mandatory to use them. Feel free to use the tools of your choice.
1) Create two EC2 Instances in AWS Cloud using,
Additional Information
 Instance Type of both instance is t2.micro
 Operating System for both instances Ubuntu Server 16.04 LTS
 Hostname of Instance 1 : MSR-test-Instance-1
 Hostname of Instance 2 : MSR-test-Instance-2
Preferred tools but not mandatory – Terraform
#############################################################################################################
we can create two instances by using Ansible roles
first off all we need to setup aws cli by using secret key and acceskey and after we can generate dynamic inventory becuse EC2 instances are having dynamic ips
vi tasks/main.yml
 tasks:
  - name: Launch instances
    ec2:
      access_key: "{{ ec2_access_key }}"
      secret_key: "{{ ec2_secret_key }}"
      keypair: "{{ ec2_keypair }}"
      group: "{{ ec2_security_group }}"
      type: "{{ ec2_instance_type }}"
      image: "{{ ec2_image }}"
      region: "{{ ec2_region }}"
      instance_tags: "{'ansible_group':'os', 'type':'{{ ec2_instance_type }}', 'group':'{{ ec2_security_group }}'}"
      count: "{{ ec2_instance_count }}"
      wait: true
    register: ec2
vi vars/main.yml
ec2_access_key: Access Key
ec2_secret_key: secret Key
ec2_region: us-east-1
ec2_zone: N.virginia
ec2_image: ami-036ede09922dadc9b 
ec2_instance_type: t2.micro
ec2_keypair: mykey
ec2_security_group: default
ec2_instance_count: 2
ec2_tag: demo
ec2_tag_name_prefix: test 
ec2_hosts: all
wait_for_port: 22
vi main.yml
---
- hosts: localhost (where the task can be defined )
  connection: local
  gather_facts: False (details of system)
 roles:
 - tasks
 - vars
:wq!
ansible-playbook main.yml
###############################################################################################
2) Once these two servers are provisioned, ensure the below following software packages are installed using
configuration management tool in both the provisioned instances.
Additional Information
 NVM – Version 0.33.2
 Node – 8.12.0
 Docker – 18.06 or latest
 Docker Compose – 1.13 or latest
 Openssl – latest version
 Git – latest version
Preferred tools – Chef / Puppet / Salt stack / Ansible.
#################################################################################################
sol: By using Ansible install the above packages
first of all add these two instance hostnames in hosts file, these two instance hostnames take it as one group
vi /etc/ansible/hosts
[msr]
MSR-test-Instance-1
MSR-test-Instance-2
:wq!
vi services.yml
---
- hosts: msr
 remote_user: ansible
 become: yes
 become_method: sudo
 connection: ssh
 gather_facts: no
tasks:
 -name: install the multipule services
  yum: name={{items}} state=latest (we can read the variables in ansible {{}}. we can install multipule sevices with the help of "itmes")
  with_items:
  - docker
  - openssl
  - docker-compose
  - git
 -name: install the services with specific version
  yum: name=nvm-0.33.2 state=present
  yum: name=node-8012.0 state=present
:wq!
ansible-playbook services.yml
#################################################################################################

3) Create a Docker Container in MSR-test-Instance-1 using Docker Compose file and ensure apache webserver is
installed. Try to use configuration management tools to automate the entire installation of apache and deploy a
sample html file from a GitHub repository.
Additional Information
 You can create your own GitHub repository with a sample html file.
Preferred tools – Chef / Puppet / Salt stack / Ansible. (Note – Ansible is Preferred)
#################################################################################################
vi docker-compose.yml
httpd:
 image: "httpd:latest"
 hostname: MSR-test-Instance-1
 ports:
   - 80:80
docker-compose up -d
vi webserver.yml
---
- hosts: MSR-test-Instance-1
  remote_user: ansible
  become_method: sudo
  tasks:
  -name: installation of apache
   yum: name=httpd state=present
  notify:
  -name: start the service
  handlers:
  -name: start the service
   service: name=httpd state=started enabled=yes
  -name deploy a sample html file from github repository
   get_url: https://github.com/karunasree4/index.git dest: /var/www/html
#####################################################################################################

4) Create a Docker Container in MSR-test-Instance-2 using Docker Compose file and ensure CouchDB Database is
installed. Try to use any configuration management tool to automate the entire installation processes.
Additional Information
 We should be able to access the Futon – web GUI of CouchDB, from the external system.
Preferred tools – Chef / Puppet / Salt stack / Ansible. (Note – Ansible is Preferred)
######################################################################################################
vi docker-compose.yml
couchdb: (container name)
  image: "couchdb:latest" (if you want to mention as image name as particular version instead of latest)
  hostname: MSR-test-Instance-2 (particular instance)
  environment: (environmental variables to access the database with help of user name and password)
     COUCHDB_USER: admin
     COUCHDB_PASSWORD: password
  ports:
     - "5984:5984"
:wq!
docker-compose up -d launching the container as a background process
#####################################################################################################
5) Commit all the code/files to GitHub and write your explanations and documentations into the readme.
    git pull origin master or
    git clone https://github.com/karunasree4/index.git
    vi README.md
    pate the above tasks
    :wq!
    git add .
    git status
    git commit -m "updated README.md
    git push origin master
6) It will be an added advantage if you can draft the step by step procedure in performing the above activities and
how to execute the code.
