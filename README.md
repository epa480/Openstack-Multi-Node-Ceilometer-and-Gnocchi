# Openstack-Multi-Node-Ceilometer-and-Gnocchi-Pike
Installation and file configure of the installation Openstack Multi-node version Pike.

This guide install only module ceilometer with gnocchi. For others modules:

https://docs.openstack.org/install-guide/

# Tutorial 

https://docs.openstack.org/ceilometer/pike/install/install-base-ubuntu.html


# Package configuration (gnocchi common)
Authentication server hostname:

http://controllerdev:35357/v3/

No configure automatically Keystone endpoint catalog

No configure automatically Database 

# Permissions

Alter permission folder /etc/gnocchi

sudo chmod 777 /etc/gnocchi/*


Configure /etc/apache2/sites-available/gnocchi-api.conf

Tutorial in construction!!!!!!



# Other bugs solved 

https://ask.openstack.org/en/question/111036/gnocchi-ceilometer-ubuntu-pike/

