# Instance Roles directory

Contains all the roles that install software on instances/VMs. These roles are applied in a pull-based manner (e.g. downloaded from a reachable object store, and applied locally to each individual instance/VM).

Roles that configure the network infrastructure and/or services go into [roles](../roles). These roles are applied ina push-based manner (Ansible's default).