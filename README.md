# LAMP stack deployment to AWS using Ansible

Using Ansible automation we will 
* Provision AWS instance
* Deploy LAMP stack on the EC2 instance
* Terminate AWS instance

In the end we will have AWS VM and LAMP stack on it.
## Prerequisites
Before you begin, ensure you have met the following requirements:
* You have [installed Ansible](https://docs.ansible.com/ansible/latest/installation_guide/intro_installation.html) with boto
* You have installed pip, boto
* You have AWS account (free tier would work as well)
* You have secure key-pair (*.pem) file from AWS.

## Installation
To install <lamp_aws_ansible_deployment>  project

On Linux:
* Copy or clone the repo
```bash
git clone https://github.com/arm-g/lamp_aws_ansible_deployment.git
```
* Get key-pair from your amazon and put it in `.boto` file in this project root
* Go to `provision_ec2_instance.yml` file and edit these vars
```
vars:
    keypair: webserver
    instance_type: t2.micro
    image: ami-07c1207a9d40bc3bd
    region: us-east-2
    group: ansible
```
* Go to `lamp_deployment/vars` subdirectory and edit `default.yml` configs if default won't work for you

The project directory should look like this. ! <hosts> file you will have in the next steps.

```
├── <hosts>
├── inventory
│   ├── ec2.ini
│   └── ec2.py
├── lamp_deployment
│   ├── files
│   │   ├── apache.conf.j2
│   │   └── info.php.j2
│   ├── playbook.yml
│   └── vars
│       └── default.yml
├── LICENSE
├── provision_ec2_instance.yml
├── README.md
├── terminate_ec2_intance.yml
└── <key-pair>.pem

```

## Usage
1) To create AWS instance run following command
```bash
ansible-playbook provision_ec2_instance.yml
```
2) After successfully finishing, the command should return provisioned VM's `IP`. Create `hosts` file and copy that ip.
Should look like this
```bash
[web-server]
<returned_vm_ip> ansible_user= <user>

[web-server:vars]
ansible_user= <ec2-user>
ansible_ssh_private_key_file= <path_to_key_pair_pemfile>
```
To make sure that VM is created run this
```bash
ansible -i hosts web-server -m ping
```
Expected result
```
<returned_vm_ip> | SUCCESS => {
    "ansible_facts": {
        "discovered_interpreter_python": "/usr/bin/python3"
    }, 
    "changed": false, 
    "ping": "pong"
}
```
3) Run
```bash
ansible-playbook -i hosts lamp_deployment/playbook.yml
```
4) Go to `http://<returned_vm_ip>` you need to see the folowing message if the deployment is done successfully
```
PHP - MYSQL Connection successfully initiated
Congrats! LAMP stack is successfully installed
```
5) To terminate the created VM and remove security group created by us we need to get VM's id (`ec2_id`). ([About dynamic inventories](https://docs.ansible.com/ansible/latest/user_guide/intro_dynamic_inventory.html))
```bash
chmod 755 ./inventory/ec2.py
./inventory/ec2.py
```
This will return JSON containing all ov your VMs. Get proper `ec2_id`. Open 
`terminate_ec2_intance.yml` and edit `vars`
```
vars:
    name: ansible
    region: us-east-2
    instance_ids:
        - 'i-xxxxxxxxxxx'
```
Then run
```bash
 ansible-playbook terminate_ec2_intance.yml
```

## Notes
* Used [lamp_ubuntu1804](https://github.com/do-community/ansible-playbooks/tree/master/lamp_ubuntu1804) git repo for LAMP deployment with some changes (added php-mysql connection healthcheck).
* The project is done in a hurry (4-5 hours limitation) so some best practices misused

## License
[MIT](https://choosealicense.com/licenses/mit/)
