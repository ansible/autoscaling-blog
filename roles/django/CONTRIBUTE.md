##Contribution

This role is designed to be used with the [Ansible Galaxy](https://galaxy.ansible.com/) project, but can also work independently.  Details on how to use Ansible Galaxy can be found on the project's website, but to install a role to your ansible controller you would run:

	ansible-galaxy install jmartin.graphite

 What's an role?  How do I use them? Check out[ this link](http://docs.ansible.com/playbooks_roles.html#roles).


If you wish to tweak this role yourself, you'll need to name your local repo the same way Ansible Galaxy would because module dependencies are mapped with Ansible Galaxy's naming scheme.

Place your roles in a role directory.  Make sure that role directory is specified in 
$HOME/.ansible.cfg, default is `/etc/ansible/roles`.

	[defaults]
	roles_path = /path/to/my/roles

Example:

* official repo name: ansible-graphite
* galaxy name: jmartin.graphite

___
	cd /path/to/my/roles
	git clone https://github.com/basho/ansible-graphite jmartin.graphite


