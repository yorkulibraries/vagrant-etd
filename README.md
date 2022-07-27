Setup a [ETD](https://github.com/yorkulibraries/etd) instance quickly with Vagrant and Ansible. The Ansible playbooks can also be used to deploy to a real production server in a similar fashion.


## Getting started

The following steps to provision a *development* instance of ETD.  

Clone this repo:
```
git clone https://github.com/yorkulibraries/vagrant-etd.git
```

Change into the vagrant-etd directory:
```
cd vagrant-etd
```

Clone the [ansible-rails](https://github.com/yorkulibraries/ansible-rails) project:
```
git clone https://github.com/yorkulibraries/ansible-rails.git
```

Clone the ETD project for development: (**NOTE:** we use SSH here to clone the ETD project because we want to be able to make changes.)
```
git clone git@github.com:yorkulibraries/etd.git
```

Install the Ansible dependencies.

```
ansible-galaxy install -r requirements.yml
```

Bring up the box with RAILS_ENV=development (this is the default):

```
vagrant up
```

Watch for any error/failed tasks. If all is good then the instance is ready to use for testing.

Apache auth_basic is used for Basic Authentication. The default users for the 3 roles: Admin, Manager, and Staff are:

```
admin/demo
manager/demo
staff/demo
```

Edit /etc/hosts and add an entry like followed so you can access the app from a browser at http://etd.me.ca/

```
192.168.168.168 etd.me.ca
```

## Verifying emails are sent in development

In development environment, mails are "caught" by MailCatcher. You can see all the emails by going to the MailCatcher web interface at http://etd.me.ca:1080/

## Set search API keys

Reserves can search Worldcat and Alma/PrimoVE for bibliographic records. To make this work, you will need API keys for each of these services.

You can create a file with the search API keys (see vars/api_keys.yml for example), then run the playbook set_api_keys.yml to set them in the Reserves database.

Note you must specify the rails_env variable to be the same as the one you have provisioned the box with.

If the file is encrypted, you will need to specify the Ansible Vault password. For example, at YUL, the file vars/yul_keys.yml is encrypted, you will run the following:

```
ansible-playbook set_api_keys.yml -e"app=reserves rails_env=development apikeys=vars/yul_keys.yml" --ask-vault-pass
```

If the file is **not** encrypted and your API keys file is var/my_keys.yml, you will run the following:

```
ansible-playbook set_api_keys.yml -e"app=reserves rails_env=development apikeys=vars/my_keys.yml"
```

## Making changes

**NOTE: Assuming you have provisioned the box with the default RAILS_ENV=development.**

The directory **/home/etd/etd** in the box is a *symlink* to **/vagrant/etd**, which is a synced folder in the your local machine's **vagrant-etd** folder.
You can make changes on your local machine in **vagrant-etd/etd** folder and it is changed in the vagrant box too. 

## Running unit tests

**NOTE: Assuming you have provisioned the box with the default RAILS_ENV=development.**

SSH into the box as user **etd**
```
ssh etd@127.0.0.1 -p2222
cd etd
RAILS_ENV=test bundle exec rake db:reset
RAILS_ENV=test bundle exec rake test
```

## Provision a vagrant box with RAILS_ENV=production

If you want to bring the box up with RAILS_ENV=production, then specify the environment variable when you run vagrant up:

```
RAILS_ENV=production vagrant up
```

## Provisioning ETD on a remote server/VM

Assuming you have a remote server dedicated for running ETD. And suppose there is a DNS record for the server such as **etd.yourdomain.ca**.

The following steps will install/configure MYSQL and ETD on that server.

Create an **inventory** file with the name/IP address of the remote server, similar to the one below:
```
me    ansible_host=192.168.168.168

[rails_app_servers]
me
```

Install Mysql and ETD on the target server

```
ansible-playbook -i inventory app_provision.yml -e"rails_env=production app_domain=yourdomain.ca mysql_root_password=db_root_password mysql_host=localhost mysql_user=etd mysql_password=etd_db_password" --limit me 
```

## About ETD
Take a look at [ETD](https://github.com/yorkulibraries/etd) repo for ETD code.
