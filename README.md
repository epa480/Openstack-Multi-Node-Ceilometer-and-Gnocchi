# Openstack-Multi-Node-Ceilometer-and-Gnocchi-Pike
Installation and file configure of the installation Openstack Multi-node version Pike in Operational System Ubuntu.

This guide install only module ceilometer with gnocchi. For others modules:

https://docs.openstack.org/install-guide/

# Install and configure for Ubuntu

## Prerequisites
Before you install and configure the Telemetry service, you must configure a target to send metering data to. The endpoint used is gnocchi.

1. Source the **admin** credentials to gain access to admin-only CLI commands:
``` 
$ . admin-openrc
``` 

2. To create the service credentials, complete these steps:
   - Create the **ceilometer** user:
   ``` 
   $ openstack user create --domain default --password-prompt ceilometer
   User Password:
   Repeat User Password:
   +-----------+----------------------------------+
   | Field     | Value                            |
   +-----------+----------------------------------+
   | domain_id | e0353a670a9e496da891347c589539e9 |
   | enabled   | True                             |
   | id        | c859c96f57bd4989a8ea1a0b1d8ff7cd |
   | name      | ceilometer                       |
   +-----------+----------------------------------+
   ``` 
   - Add the **admin** role to the **ceilometer** user. 
   ```
   $ openstack role add --project service --user ceilometer admin
   ```
          Note: This command provides no output.
   - Create the **ceilometer** service entity:
   ```
   $ openstack service create --name ceilometer \
   --description "Telemetry" metering
   +-------------+----------------------------------+
   | Field       | Value                            |
   +-------------+----------------------------------+
   | description | Telemetry                        |
   | enabled     | True                             |
   | id          | 5fb7fd1bb2954fddb378d4031c28c0e4 |
   | name        | ceilometer                       |
   | type        | metering                         |
   +-------------+----------------------------------+
   ```
3. Register Gnocchi service in Keystone:   
   - Create the **gnocchi** user:
   ```
   $ openstack user create --domain default --password-prompt gnocchi
   User Password:
   Repeat User Password:
   +-----------+----------------------------------+
   | Field     | Value                            |
   +-----------+----------------------------------+
   | domain_id | e0353a670a9e496da891347c589539e9 |
   | enabled   | True                             |
   | id        | 8bacd064f6434ef2b6bbfbedb79b0318 |
   | name      | gnocchi                          |
   +-----------+----------------------------------+
   ```
   - Add the admin role to the gnocchi user.
   ```
   $ openstack role add --project service --user gnocchi admin
   ```
          Note: This command provides no output.
   - Create the Metric service API endpoints:
   ```
   $ openstack endpoint create --region RegionOne \
     metric public http://controller:8041
   +--------------+----------------------------------+
   | Field        | Value                            |
   +--------------+----------------------------------+
   | enabled      | True                             |
   | id           | b808b67b848d443e9eaaa5e5d796970c |
   | interface    | public                           |
   | region       | RegionOne                        |
   | region_id    | RegionOne                        |
   | service_id   | 205978b411674e5a9990428f81d69384 |
   | service_name | gnocchi                          |
   | service_type | metric                           |
   | url          | http://controller:8041           |
   +--------------+----------------------------------+

   $ openstack endpoint create --region RegionOne \
     metric internal http://controller:8041
   +--------------+----------------------------------+
   | Field        | Value                            |
   +--------------+----------------------------------+
   | enabled      | True                             |
   | id           | c7009b1c2ee54b71b771fa3d0ae4f948 |
   | interface    | internal                         |
   | region       | RegionOne                        |
   | region_id    | RegionOne                        |
   | service_id   | 205978b411674e5a9990428f81d69384 |
   | service_name | gnocchi                          |
   | service_type | metric                           |
   | url          | http://controller:8041           |
   +--------------+----------------------------------+

    $ openstack endpoint create --region RegionOne \
      metric admin http://controller:8041
   +--------------+----------------------------------+
   | Field        | Value                            |
   +--------------+----------------------------------+
   | enabled      | True                             |
   | id           | b2c00566d0604551b5fe1540c699db3d |
   | interface    | admin                            |
   | region       | RegionOne                        |
   | region_id    | RegionOne                        |
   | service_id   | 205978b411674e5a9990428f81d69384 |
   | service_name | gnocchi                          |
   | service_type | metric                           |
   | url          | http://controller:8041           |
   +--------------+----------------------------------+
   ```

## Install Gnocchi
1. Install the Gnocchi packages. Alternatively, Gnocchi can be install using pip:
```
# apt-get install gnocchi-api gnocchi-metricd python-gnocchiclient
```
       Note:Depending on your environment size, consider installing Gnocchi separately as it makes extensive use of the cpu.
2. Create the database for Gnocchi’s indexer:
   - Use the database access client to connect to the database server as the **root** user:
   ```
   $ mysql -u root -p
   ```
   - Create the gnocchi database:
   ```
   CREATE DATABASE gnocchi;
   ```
   - Grant proper access to the gnocchi database:
   ```
   GRANT ALL PRIVILEGES ON gnocchi.* TO 'gnocchi'@'localhost' \
     IDENTIFIED BY 'GNOCCHI_DBPASS';
   GRANT ALL PRIVILEGES ON gnocchi.* TO 'gnocchi'@'%' \
     IDENTIFIED BY 'GNOCCHI_DBPASS';
   ```
   Replace **GNOCCHI_DBPASS** with a suitable password.
   - Exit the database access client.


 

       




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

