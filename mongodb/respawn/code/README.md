# Source: Re-spawn a MongoDB server automatically

This is the source code for my article ["Sleep better at night! Here is how to re-spawn a MongoDB Server automatically!"](https://github.com/sarjarapu/whitepapers/mongodb/respawn/README.md). If you haven't read that article yet, I suggest you to read through it to understand the details. Here in this article, I would be just discuss about the code I used. While I tried my best to keep it simple/generic, I could have missed a thing or two. So, feel free to fork it to make it fit for your needs or drop me a note.

Before you want to get your hands dirty with the source code, I request you to read through the [configuration](#configuration) section to help customize the settings to fit your environment.

## Introduction

I am assuming you have tools like git, [Ansible](https://www.ansible.com/)  , [AWS CLI](https://aws.amazon.com/cli/), [jq](https://stedolan.github.io/jq/) etc are installed and pre-configured. If don't haven them yet then these guides may help you to get started

-   [Installing the AWS Command Line Interface](http://docs.aws.amazon.com/cli/latest/userguide/installing.html)
-   [Configuring the AWS  CLI](http://docs.aws.amazon.com/cli/latest/userguide/cli-chap-getting-started.html)
-   [Installation of Ansible](http://docs.ansible.com/ansible/latest/intro_installation.html)
-   [Download Jq](https://stedolan.github.io/jq/download/)
-   [Getting Started - Installing Git](https://git-scm.com/book/en/v2/Getting-Started-Installing-Git)

Once you are done with the installation of these tools, please continue here to understand what configurations you need to make.

#### Configure Ansible Vault

Update the contents of Ansible
[Vault](http://docs.ansible.com/ansible/latest/vault.html)
password file located at `ansible/private/.vault_pass` with password of your
choice. The password listed in here would be used to encrypt or decrypt
the sensitive information listed in vault located at
`ansible/group_vars/all/vault`. Since anyone with access to vault_pass in source
code can decrypt the sensitive information, the `.gitignore` file is
configured to ignore any updates to vault password file.

Also update the contents of the vault file with your AWS access/secret
keys.

```
vault_aws_access_key: <aws_access_key>
vault_aws_secret_key: <aws_secret_key>
```

#### Update Ansible Configuration file

Update the file path for *private_key_file* variable in `ansible/ansible.cfg`
[configuration](http://docs.ansible.com/ansible/latest/intro_configuration.html)
file. It is the file path for pem key file that you use for [Connecting
to Your Linux Instance Using
SSH](http://docs.aws.amazon.com/AWSEC2/latest/UserGuide/AccessingInstancesLinux.html)

#### Configure Ansible Variables

Update the values for below variables in file
`ansible/roles/aws/defaults/main.yml`

```
fqdn_domain: "<domain name here>"
security_key: "<aws security key name>"
security_group: "<aws security group name>"
```

Update these below set of values to further customize the hardware
configuration for your MongoDB deployment.

```
region: "us-west-2"
instance_type: "t2.micro"
data_vol_size: <volume size in GB>
ami: "<ami id of your choice>"
group_name: "<your team / organization group name>"
app_name: "<application name>"
```

### Source code

Finally, here are a few Ansible scripts to help you

- Create VPC with public/private subnet
- Provision EC2 in AWS
- Create / configure MongoDB replicaset
- Create Auto Scaling group

```
# clone the GitHub repo
git clone git@github.com:sarjarapu/whitepapers.git
cd whitepapers/mongodb/respawn/code/ansible

# create vpc environment
ansible-playbook 00.create.vpc.yml --extra-vars
'network_name=<your_appname> domain_name <your_domain_name>'

# provision the hardware
ansible-playbook 01.provision.yml

# create a replica set on the above provisioned servers
ansible-playbook 02.replica set.yml -i
inventories/<server_group_name> --extra-vars
'{"replset_name":"<replica set_name>","mongo_port":<port>}'

# install the required tools and packages on above servers
ansible-playbook 03.install.utils.yml -i
inventories/<server_group_name>
--extra-vars='source_pem_key_file=<aws_pem_key_filepath>'

# create auto scaling group
ansible-playbook 04.create.auto.scale.group.yml -i
inventories/<server_group_name> --extra-vars='vpc_zone_identifier=<your_subnet_id>'

# have a cron job that runs on every member in replica set
ansible-playbook 10.check.mounts.yml -i
inventories/<server_group_name>
```
