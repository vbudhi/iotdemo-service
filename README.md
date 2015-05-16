#### An Ambari Stack for HDP IoT Demo
Ambari stack for easily installing and managing HDP IoT Demo which shows real time monitoring/alerts and predictions of driving violations generated by fleet of trucks. 
Pre-reqs: 
  - You must have access to [sedev git repo](https://github.com/hortonworks/sedev) to install this service. 
  - The service currently requires that it is installed on the Ambari server node and that Kafka and Zookeeper and also running on the same node.
  - HBase and Storm must be available on the cluster and started. 
  - Falcon must be stopped before installing this service.


##### Setup steps

- Download HDP 2.2.4.2 sandbox VM image (Sandbox_HDP_2.2.4.2_VMWare.ova) from [Hortonworks website](http://hortonworks.com/products/hortonworks-sandbox/)
- Import Sandbox_HDP_2.2.4.2_VMWare.ova into VMWare and set the VM memory size to 8GB
- Now start the VM
- After it boots up, find the IP address of the VM and add an entry into your machines hosts file e.g.
```
192.168.191.241 sandbox.hortonworks.com sandbox    
```
- Connect to the VM via SSH (password hadoop) and restart Ambari server
```
ssh root@sandbox.hortonworks.com
/root/start_ambari.sh
```

- **Make sure Storm, HBase, Kafka, Hive are up and Falcon is down** and all these services are out of maintenance mode. You can run below from Ambari server as a shortcut

```
#Ambari password
export PASSWORD=admin
#Ambari host
export AMBARI_HOST=localhost

#detect name of cluster
output=`curl -u admin:$PASSWORD -i -H 'X-Requested-By: ambari'  http://$AMBARI_HOST:8080/api/v1/clusters`
CLUSTER=`echo $output | sed -n 's/.*"cluster_name" : "\([^\"]*\)".*/\1/p'`

#make sure kafka, storm, falcon are out of maintenance mode
curl -u admin:$PASSWORD -i -H 'X-Requested-By: ambari' -X PUT -d '{"RequestInfo": {"context" :"Remove Falcon from maintenance mode"}, "Body": {"ServiceInfo": {"maintenance_state": "OFF"}}}' http://$AMBARI_HOST:8080/api/v1/clusters/$CLUSTER/services/FALCON
curl -u admin:$PASSWORD -i -H 'X-Requested-By: ambari' -X PUT -d '{"RequestInfo": {"context" :"Remove Kafka from maintenance mode"}, "Body": {"ServiceInfo": {"maintenance_state": "OFF"}}}' http://$AMBARI_HOST:8080/api/v1/clusters/$CLUSTER/services/KAFKA
curl -u admin:$PASSWORD -i -H 'X-Requested-By: ambari' -X PUT -d '{"RequestInfo": {"context" :"Remove Storm from maintenance mode"}, "Body": {"ServiceInfo": {"maintenance_state": "OFF"}}}' http://$AMBARI_HOST:8080/api/v1/clusters/$CLUSTER/services/STORM
curl -u admin:$PASSWORD -i -H 'X-Requested-By: ambari' -X PUT -d '{"RequestInfo": {"context" :"Remove Hbase from maintenance mode"}, "Body": {"ServiceInfo": {"maintenance_state": "OFF"}}}' http://$AMBARI_HOST:8080/api/v1/clusters/$CLUSTER/services/HBASE
curl -u admin:$PASSWORD -i -H 'X-Requested-By: ambari' -X PUT -d '{"RequestInfo": {"context" :"Remove Hbase from maintenance mode"}, "Body": {"ServiceInfo": {"maintenance_state": "OFF"}}}' http://$AMBARI_HOST:8080/api/v1/clusters/$CLUSTER/services/HIVE


#Start Kafka, Storm, HBase, Hive
curl -u admin:$PASSWORD -i -H 'X-Requested-By: ambari' -X PUT -d '{"RequestInfo": {"context" :"Start KAFKA via REST"}, "Body": {"ServiceInfo": {"state": "STARTED"}}}' http://$AMBARI_HOST:8080/api/v1/clusters/$CLUSTER/services/KAFKA
curl -u admin:$PASSWORD -i -H 'X-Requested-By: ambari' -X PUT -d '{"RequestInfo": {"context" :"Start STORM via REST"}, "Body": {"ServiceInfo": {"state": "STARTED"}}}' http://$AMBARI_HOST:8080/api/v1/clusters/$CLUSTER/services/STORM
curl -u admin:$PASSWORD -i -H 'X-Requested-By: ambari' -X PUT -d '{"RequestInfo": {"context" :"Start HBASE via REST"}, "Body": {"ServiceInfo": {"state": "STARTED"}}}' http://$AMBARI_HOST:8080/api/v1/clusters/$CLUSTER/services/HBASE
curl -u admin:$PASSWORD -i -H 'X-Requested-By: ambari' -X PUT -d '{"RequestInfo": {"context" :"Start HBASE via REST"}, "Body": {"ServiceInfo": {"state": "STARTED"}}}' http://$AMBARI_HOST:8080/api/v1/clusters/$CLUSTER/services/HIVE


#stop Falcon
curl -u admin:$PASSWORD -i -H 'X-Requested-By: ambari' -X PUT -d '{"RequestInfo": {"context" :"Start HBASE via REST"}, "Body": {"ServiceInfo": {"state": "INSTALLED"}}}' http://$AMBARI_HOST:8080/api/v1/clusters/$CLUSTER/services/FALCON

```

- To deploy the IOTDEMO stack, run below
```
cd /var/lib/ambari-server/resources/stacks/HDP/2.2/services
git clone https://github.com/abajwa-hw/iotdemo-service.git    
sudo service ambari restart
```
- Then you can click on 'Add Service' from the 'Actions' dropdown menu in the bottom left of the Ambari dashboard:

On bottom left -> Actions -> Add service -> check IOTDEMO server -> Next -> Next -> Configure service -> Next -> Deploy
![Image](../master/screenshots/select-service.png?raw=true)

Things to remember while configuring the service
  - The service currently requires that it is installed on the Ambari server node and that Kafka and Zookeeper and also running on the same node.
  - Under "Advanced demo-config", you need to enter your github credentials to allow the service to access the IoT demo artifacts
  - Under "Advanced user-config", you need to update your ambari user/password/port configuration (if not using default)
  - The IoT demo configs are available under "Advanced demo-env", but do not require updating as all required configs will be auto-populated:
    - Ambari host
    - Name node host/port
    - Nimbus host
    - Hive metastore host/port
    - Supervisor host
    - HBase master host
    - Kafka host/port (also where ActiveMQ will be installed)
  
![Image](../master/screenshots/config1.png?raw=true)

![Image](../master/screenshots/config2.png?raw=true)
      

- On successful deployment you will see the IOTDEMO service as part of Ambari stack and will be able to start/stop the service from here:
![Image](../master/screenshots/started-service.png?raw=true)


- You can see the parameters you configured under 'Configs' tab
![Image](../master/screenshots/started-config.png?raw=true)

- One benefit to wrapping the component in Ambari service is that you can now monitor/manage this service remotely via REST API
```
export SERVICE=IOTDEMO
export PASSWORD=admin
export AMBARI_HOST=localhost

#detect name of cluster
output=`curl -u admin:$PASSWORD -i -H 'X-Requested-By: ambari'  http://$AMBARI_HOST:8080/api/v1/clusters`
CLUSTER=`echo $output | sed -n 's/.*"cluster_name" : "\([^\"]*\)".*/\1/p'`

#get service status
curl -u admin:$PASSWORD -i -H 'X-Requested-By: ambari' -X GET http://$AMBARI_HOST:8080/api/v1/clusters/$CLUSTER/services/$SERVICE

#start service
curl -u admin:$PASSWORD -i -H 'X-Requested-By: ambari' -X PUT -d '{"RequestInfo": {"context" :"Start $SERVICE via REST"}, "Body": {"ServiceInfo": {"state": "STARTED"}}}' http://$AMBARI_HOST:8080/api/v1/clusters/$CLUSTER/services/$SERVICE

#stop service
curl -u admin:$PASSWORD -i -H 'X-Requested-By: ambari' -X PUT -d '{"RequestInfo": {"context" :"Stop $SERVICE via REST"}, "Body": {"ServiceInfo": {"state": "INSTALLED"}}}' http://$AMBARI_HOST:8080/api/v1/clusters/$CLUSTER/services/$SERVICE
```


- To remove the IOTDEMO service: 
  - Stop the service via Ambari
  - Delete the service
  
    ```
export PASSWORD=admin
export AMBARI_HOST=localhost

output=`curl -u admin:$PASSWORD -i -H 'X-Requested-By: ambari'  http://$AMBARI_HOST/api/v1/clusters`
CLUSTER=`echo $output | sed -n 's/.*"cluster_name" : "\([^\"]*\)".*/\1/p'`
    
curl -u admin:$PASSWORD -i -H 'X-Requested-By: ambari' -X DELETE http://$AMBARI_HOST:8080/api/v1/clusters/$CLUSTER/services/IOTDEMO
    ```
  - Remove artifacts 
  
    ```
rm -rf /var/lib/ambari-server/resources/stacks/HDP/2.2/services/iotdemo*
rm -rf /root/sedev
    ```


#### Access webapp

- Open the webapp at http://sandbox.hortonworks.com:8081



