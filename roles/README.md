# Roles directory

Contains all the roles for network infrastructure and services. These roles are applied in a push-based manner (Ansible's default).

Roles that install software on instances/VMs go into [instance-roles](../instance-roles), and are applied ina a pull-based manner (e.g. downloaded from a reachable object store and applied locally to each individual instance/VM).