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



# BDR DataScience Stack

This project provides a virtual machine with some essential tools for data
science. The goal is to provide a stack which enables making maximal use of the
resources of your machine. It is not a cluster, nor is it meant to replace a
cluster.

# Spark

A spark master and a spark slave are started automatically on boot. The status
of both can be inspected through a browser via the address: [http://10.0.0.21:8080/](http://10.0.0.21:8080/)

To stop the master and/or slave:

    sudo service spark-master stop
    sudo service spark-slave stop

# RStudio

The stack provides RStudio server which gives the RStudio interface in the
browser. For now:

  url: 10.0.0.21:8787
login: vagrant / vagrant.

To use spark initialize a spark context as follows
    
    > library(SparkR)
    > sc <- sparkR.init(master="spark://10.0.0.21:7077")
    
# Jupyter

Jupyther is installed and starts automatically on boot. The notebook directory
for jupyter is set to /home/data/science/notebooks in the vm. Notebooks can be
accessed through a browser via the address: [http://10.0.0.21:8888](http://10.0.0.21:8888)
