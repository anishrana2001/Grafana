# **Grafana Installation on Container**

### Visit the Official website of Grafana
```
https://grafana.com/
```
### Click on "Open Source" then again click on "Docs"

### Let's create the Grafana Container
```
docker run -d -p 3000:3000 --name=grafana grafana/grafana-oss
```
### Check the status of container
```
docker ps 
```
### Check the IP adddress of your VM
```
ip a
```
### Default Credential is 
```
admin/admin
```
### Check the Container ID or name

```
docker ps 
```
### How to stop the Container
```
docker stop grafana
```
### Check the container
```
docker ps 
```
### How to start the container?
```
docker start grafana
```
### Again stop the container
```
docker stop grafana
```
### Check the container again
```
docker ps 
```
### Check the container - You should use "-a" option
```
docker ps -a
```
### How to remove the Grafana container?
```
docker rm grafana
```
```
docker ps -a
```


### Recommendations are using Volume:

### create a persistent volume for your data
```
docker volume create grafana-storage
```
### verify that the volume was created correctly you should see some JSON output
```
docker volume inspect grafana-storage
```
### start grafana
```
docker run -d -p 3000:3000 --name=grafana \
  --volume grafana-storage:/var/lib/grafana \
  grafana/grafana-oss
```

```
docker ps 
```
```
docker rm -f grafana
```

### Use Bind path :
#### create a directory for your data
```
mkdir data
```
### start grafana with your user id and using the data directory
```
docker run -d -p 3000:3000 --name=grafana \
  --user "$(id -u)" \
  --volume "$PWD/data:/var/lib/grafana" \
  grafana/grafana-oss
```
```
docker ps 
```
```
docker rm -f grafana
```

```
docker volume rm  grafana-storage
```

We can also specify the Variables:
```
docker run -d -p 3000:3000 --name=grafana -e "GF_LOG_LEVEL=debug" grafana/grafana-oss
```


### If one wants to modify the configuration files of Grafana, then it may use the environment variables. 
### https://grafana.com/  --> Click on "Open Source" then again click on "Docs"  --> "Set up" --> "Configure Grafana"
```
GF_<SECTION NAME>_<KEY>
```

#### In the "ini" file, 
```
[security]
admin_user = admin
```

### Then, our variable would be = GF_security_admin_user

```
feature_toggles
```
```
docker run -d -p 3000:3000 --name=grafana -e "GF_FEATURE_TOGGLES_ENABLE=publicDashbord" grafana/grafana-enterprise
```

### Thus, we can say that environment variables are the clean process to update the configuration of Grafana.
![image](https://github.com/user-attachments/assets/ebdf5d9a-08c7-4ff9-b364-e2e7d96c05c8)



### To know all plugins for Grafana:
```
https://grafana.com/grafana/plugins/
```
### Zabbix

### Infinity 


#### Core / community 

### Community : It is not being managed by Grafana and it directly manage by community only. 
### Core: Grafana packages, and Grafana is responsible.


### Data Source is also a Plugins, which tells us that the data is coming from standard sources
