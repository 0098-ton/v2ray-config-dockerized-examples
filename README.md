# dockerized setup of v2ray vmess with obfuscated headers 

in this setup we are going to configure a v2ray server using docker that accepts obfuscated vmess connections as inbound and route it to freedom (internet) as outbound,

then in client we will run a v2ray proxy node that accepts shadowsocks, http_proxy and dokodemo-door(accepts any connection) as inbound 

and route inbounds to our v2ray vmess server as outbound , client setup can be done in dockerized way or using [v2ray core](https://github.com/v2ray/v2ray-core).

### server config:

 1. install [docker](https://docs.docker.com/engine/install/ubuntu/) with compose plugin
 2. clone the repo and cd into server directory
 ```
 $ git clone https://github.com/kayvan-eth/v2ray-config-dockerized-examples.git && cd v2ray-config-dockerized-examples/server
 ```

 3. install uuid and create one for v2ray client,
 ```
 $ apt install -y uuid && uuid -v 4
 3ef0f508-4b96-44a6-95d7-6dea91df5eb3
 ```
 and replace the output ( 3ef0f508-4b96-44a6-95d7-6dea91df5eb3 ) with "UUID_V_4" in server/v2config.json file. (keep this id ,we will need it for client configuration)

 4. start the service :
 ```
 docker compose up -d && docker ps
 CONTAINER ID   IMAGE                                  COMMAND                  CREATED       STATUS       PORTS                                                                  NAMES
 c66862df4107   v2ray/official                         "v2ray -config=/etc/…"   5 weeks ago   Up 3 weeks   0.0.0.0:1433->1433/tcp, 0.0.0.0:1433->1433/udp                         v2ray
 ```

### client config:

#### setup using docker

 1. install [docker](https://docs.docker.com/engine/install/ubuntu/) with compose plugin
 2. clone the repo and cd into client directory
 ```
 $ git clone https://github.com/kayvan-eth/v2ray-config-dockerized-examples.git && cd v2ray-config-dockerized-examples/client
 ```
 3. edit v2config,json, replace "SERVER_PUBLIC_IP" with your server public ip and "UUID_V_4" with the one you generated before in step 3 of server setup(3ef0f508-4b96-44a6-95d7-6dea91df5eb3) .

 4. start services with command below:
 ```
 $ docker compose up -d && docker ps
 CONTAINER ID   IMAGE            COMMAND                  CREATED       STATUS        PORTS                                                                                        NAMES
 2a454b4cd5ca   v2ray/official   "v2ray -config=/etc/…"   5 weeks ago   Up 34 hours   0.0.0.0:443->443/tcp, 0.0.0.0:443->443/udp, 0.0.0.0:2080->2080/tcp, 0.0.0.0:2080->2080/udp, 0.0.0.0:22233->22233/tcp, 0.0.0.0:22233->22233/udp v2ray
 ```
## OR  
#### setup using v2ray-core

 1. download [v2ray binary ](https://github.com/v2fly/v2ray-core/releases/tag/v4.31.0) based on your os extract it and put it in PATH.
 
 2. clone the repo and cd into client directory
 ```
 $ git clone https://github.com/kayvan-eth/v2ray-config-dockerized-examples.git && cd v2ray-config-dockerized-examples/client
 ```
 3. edit v2config,json, replace "SERVER_PUBLIC_IP" with your server public ip and "UUID_V_4" with the one you generated before in step 3 of server setup(3ef0f508-4b96-44a6-95d7-6dea91df5eb3) .

 4. start v2ray with command below:
 ```
 $ v2ray -c PATH_TO_V2CONFIG_JSON
 ```
 NOTE: if you are running on wsl, run it with sudo for better port mapping

 now you have a ready v2ray client instance that accepts shadowsocks on port 443 , http-proxy on port 2080 and a dokodemo-door on port 22233.

 ## tips and v2ray config explanation:

 ### CAUTION: if you did edit the v2config anywhere dont forget to restart the v2ray docker.

 for changing inbounds configuration and credentials , simply just edit client/v2cnfig.json inbounds section.
 for example in this setup http-proxy url will be http://user:password@V2RAY_CLIENT_IP:2080 and V2RAY_CLIENT_IP can be 127.0.0.1, localhost or any ip that you see the v2ray core.

 ```
        {
            "listen": "0.0.0.0",
            "port": 2080,
            "protocol": "http",
            "settings": {
                "udp": true,
                "accounts": [
                    {
                        "user": "user",
                        "pass": "password"
                    }
                ]
            }
        } 
 ```

 by defaut in this setup for client , shadowsock http and dokodemo are enabled, you can disable any of them by deleting their configuration in client/v2config.json

 ```
     "inbounds": [
        {
            "listen": "0.0.0.0",
            "port": 2080,
            "protocol": "http",
            "settings": {
                "udp": true,
                "accounts": [
                    {
                        "user": "user",
                        "pass": "password"
                    }
                ]
            }
        },
### for example delete this section if you want to disable shadowsocks interface
        {
            "port": 443,
            "protocol": "shadowsocks",
            "settings": {
                "method": "aes-256-gcm",
                "password": "password",
                "level": 1
            }
        },
##########################################################################33
        {
          "tag":"any",
          "port":22233,
          "protocol":"dokodemo-door",
          "network": "tcp,udp",
          "timeout": 30,
          "followRedirect": true
        }
    ]
 ```

NOTE: if you changed the v2onfig inbound ports , dont forget to change the compose file as it needs.

## add more peers:

for adding new clients , on the server, create another uuid: 
```
$ uuid -v 4
146394c5-a9ec-4a78-a3ae-fb28f58d0b5f
```
and add the output in v2ray v2config.json like below:
```
            "clients": [
                {
                    "id": "3ef0f508-4b96-44a6-95d7-6dea91df5eb3",
                    "alterId": 233,
                    "security": null
                },
                {
                    "id": "146394c5-a9ec-4a78-a3ae-fb28f58d0b5f",
                    "alterId": 234,
                    "security": null
                }
```

NOTE: dont forget to pick unique alterId for clients and also it must be the same in client and config for each client

## ALSO

there are more ways to achieve this goal by configuring the v2ray docker client on a seperate machine from clients,

then add more http-proxy , shadowsocks, ... on the v2ray client and assign them to users

for http-proxy:
```
        {
            "listen": "0.0.0.0",
            "port": 1080,
            "protocol": "http",
            "settings": {
                "udp": true,
                "accounts": [
                    {
                        "user": "user",
                        "pass": "password"
                    },
                    {
                        "user": "user2",
                        "pass": "password2"
                    },
                    .
                    .
                    .
                ]
            }
        },

```
also for http-proxy you can have multiple interface by adding:

```
        {
            "listen": "0.0.0.0",
            "port": 2080,
            "protocol": "http",
            "settings": {
                "udp": true,
                "accounts": [
                    {
                        "user": "user",
                        "pass": "password"
                    }
                ]
            }
        },
        {
            "listen": "0.0.0.0",
            "port": 2081,
            "protocol": "http",
            "settings": {
                "udp": true,
                "accounts": [
                    {
                        "user": "user",
                        "pass": "password"
                    }
                ]
            }
        },
        .
        .
        .
```
NOTE: dont forget to assign unique ports to multiple interfaces.

and for shadowsocks:
```
        {
            "port": 443,
            "protocol": "shadowsocks",
            "settings": {
                "method": "aes-256-gcm",
                "password": "password",
                "level": 1
            }
        },
        {
            "port": 444,
            "protocol": "shadowsocks",
            "settings": {
                "method": "aes-256-gcm",
                "password": "password2",
                "level": 1
            }
        },

```
NOTE: dont forget to assign unique ports to multiple interfaces.

