# Openstack-Multi-Node-Ceilometer-and-Gnocchi-Pike
Installation and file configure of the installation Openstack Multi-node version Pike in Operational System Ubuntu.

This guide install only module ceilometer with gnocchi(controller service). For others modules:

https://docs.openstack.org/install-guide/
https://docs.openstack.org/install-guide/openstack-services.html#minimal-deployment

For the compute service , install the official documentation. It's working :+1:

https://docs.openstack.org/ceilometer/pike/install/install-compute-ubuntu.html

For other controller services(Cinder,Glance,Heat,Keystone,Neutron and Swift. It's working :+1:

https://docs.openstack.org/ceilometer/pike/install/


# Install and configure for Ubuntu Controller Node

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
   - Create the gnocchi service entity:
   ```
   $ openstack service create --name gnocchi \
   --description "Metric Service" metric
   
      - Add the admin role to the gnocchi user.
   ```
   $ openstack role add --project service --user gnocchi admin
   ```
          Note: This command provides no output.
    
    ``` 
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
   - Package configuration (gnocchi common)
   
       Setup a database for gnocchi:
       - [ ] Yes
       - [x] No
       
       Authentication server hostname:
       - http://controller:35357/v3/
       
       Register Gnocchi in the Keystone endpoint catalog 
       - [ ] Yes
       - [x] No
       
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
3. Edit the **/etc/gnocchi/gnocchi.conf** file and add Keystone options:  
   - In the **[api]** section, configure gnocchi to use keystone:
   ```
   [api]
   auth_mode = keystone
   ```
   - In the [keystone_authtoken] section, configure keystone authentication:
   ```
   [keystone_authtoken]
   ...
   auth_type = password
   auth_url = http://controller:5000/v3
   project_domain_name = Default
   user_domain_name = Default
   project_name = service
   username = gnocchi
   password = GNOCCHI_PASS
   interface = internalURL
   region_name = RegionOne
   ``` 
   Replace **GNOCCHI_PASS** with the password you chose for the gnocchi user in the Identity service.
   
   - In the **[indexer]** section, configure database access:
   ```
   [indexer]
   url = mysql+pymysql://gnocchi:GNOCCHI_DBPASS@controller/gnocchi
   ```
   Replace **GNOCCHI_PASS** with the password you chose for the gnocchi user in the Identity service.
   
   - In the **[storage]** section, configure location to store metric data. In this case, we will store it to the local file system.       
   See Gnocchi documenation for a list of more durable and performant drivers:
   ```
   [storage]
   coordination_url = file:///var/lib/gnocchi/locks
   file_basepath = /var/lib/gnocchi
   driver = file
   ```
4. Change Permission directoty **/etc/gnocchi/ **:  
   ```
   # chmod 777 /etc/gnocchi/*
   ```
5. Alter configuration file apache2 **gnocchi-api.conf** :
   This part was retired [StackOverFlow Page](https://stackoverflow.com/questions/45374863/devstackceilometergnocchi-error-403).
   Open the file: 
   ```
   # geany /etc/apache2/sites-available/gnocchi-api.conf
   # service apache2 restart
   ```   
   Replace the contents of the file with the github file [gnocchi-api.conf](https://github.com/pablobrunetti/Openstack-Multi-Node-Ceilometer-and-Gnocchi/blob/master/gnocchi-api.conf)
6. Initialize Gnocchi:  
   ```
   $ gnocchi-upgrade
   ```
## Finalize Gnocchi installation
1. Restart the Gnocchi services:
    ```
    # service gnocchi-api restart
    # service gnocchi-metricd restart
    ```  
## Install and configure components
1. Install the ceilometer packages:
    ```
    # apt-get install ceilometer-agent-notification \
      ceilometer-agent-central
    ``` 
  
2. Edit the **/etc/ceilometer/ceilometer.conf** file and complete the following actions:   
   - Configure Gnocchi connection:
     ```
     [dispatcher_gnocchi]
     # filter out Gnocchi-related activity meters (Swift driver)
     filter_service_activity = False
     # default metric storage archival policy
     archive_policy = low
     ```
   - In the **[DEFAULT]** section, configure **RabbitMQ** message queue access:
     ```
     [DEFAULT]
     ...
     transport_url = rabbit://openstack:RABBIT_PASS@controller
     ```
     Replace **RABBIT_PASS** with the password you chose for the **openstack** account in **RabbitMQ**.
   - In the **[service_credentials]** section, configure service credentials: 
     ```
     # ceilometer-upgrade --skip-metering-database
     ```
## Finalize installation
1. Restart the Telemetry services:
   ```
   # service ceilometer-agent-central restart
   # service ceilometer-agent-notification restart
   ```
##Testing Application [Verify](https://docs.openstack.org/ceilometer/pike/install/verify.html). 

Run:
``` 
$ export OS_AUTH_TYPE=password 
```

Test gnocchi with command: 
``` 
$ gnocchi metric list
```
A list of metrics will appear in bash
   

## Enable Metrics Hardware gnocchi

https://ask.openstack.org/en/question/105993/how-to-enable-hardware-metrics-on-gnocchi/














