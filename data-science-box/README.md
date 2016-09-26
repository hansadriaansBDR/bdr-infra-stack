# BigData Republic's Data Science Box


## Prerequisites

* VirtualBox (VMware shared folder to host is not supported!)
* Vagrant 


## Usage


In order to enable shared folders with the host that are bidirectionally synced, virtualbox guest additions should stay up to date (only run this once):
```
vagrant plugin install vagrant-vbguest
```

To start the box:
```
vagrant up
```

To suspend the box:

```
vagrant suspend
```

To uninstall the box:

```
vagrant destroy
```

After fully provisioned the Jupyter notebook should be available at http://10.0.0.42
