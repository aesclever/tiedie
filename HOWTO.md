# How to scalfold the tiedie gateway

*By* **Mark Nguyen**

The followings are based on forked git repo on https://github.com/aesclever/tiedie

Original source: https://github.com/iot-onboarding/tiedie 


## Setting up `docker compose`

```
sudo apt-get update
sudo apt-get install ca-certificates curl
sudo install -m 0755 -d /etc/apt/keyrings
sudo curl -fsSL https://download.docker.com/linux/ubuntu/gpg -o /etc/apt/keyrings/docker.asc

sudo chmod a+r /etc/apt/keyrings/docker.asc
# Add the repository to Apt sources:
echo   "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.asc] https://download.docker.com/linux/ubuntu \
  $(. /etc/os-release && echo "$VERSION_CODENAME") stable" |   sudo tee /etc/apt/sources.list.d/docker.list > /dev/null

sudo apt-get update
sudo apt-get install docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin
sudo systemctl start docker
sudo systemctl status docker
sudo docker compose up --build
```

## Setting up the `tiedie` gateway

1.  Clone the repo

```
https://github.com/aesclever/tiedie.git
```
2.  Configure via instructions in [README](https://github.com/aesclever/tiedie/blob/main/gateway/README.md)

3.  Run the docker compose command to start the tiedie gateway container
```
sudo docker compose up --build
```

## Troubleshoot gateway issues

You might run into issue of IoT detection:

```
[+] Running 1/5                                                                        
 ✔ Network gateway_default        Created                                         0.8s 
 ⠸ Container gateway-adminer-1    Created                                         8.3s 
 ⠸ Container gateway-postgres-1   Created                                         8.3s 
 ⠙ Container gateway-mosquitto-1  Created                                         3.1s 
 ⠦ Container gateway-tiedie-ap-1  Created                                         1.6s 
Attaching to adminer-1, mosquitto-1, postgres-1, tiedie-ap-1
mosquitto-1  | 1713164360: Loading config file /etc/mosquitto/conf.d/go-auth.conf
mosquitto-1  | 1713164360: mosquitto version 2.0.15 starting
mosquitto-1  | 1713164360: Config loaded from /etc/mosquitto/mosquitto.conf.
mosquitto-1  | 1713164360: Loading plugin: /mosquitto/go-auth.so
Error response from daemon: error gathering device information while adding custom device "/dev/ttyACM0": no such file or directory
```

**Posible solution**:
- Clone https://github.com/aesclever/docker-domoticz
- Run the docker command to pass through a ttyDevice.
Consult its README for more details.
