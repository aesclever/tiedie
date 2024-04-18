# How to scalfold the tiedie gateway

*By* **Mark Nguyen**

The followings are based on forked git repo on https://github.com/aesclever/tiedie

Original source: https://github.com/iot-onboarding/tiedie 

**Note**  Since this is a forked repo, some programmed references to the original source might fail.  You should know where to look in these cases.

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
Register with Silabs and download the bluetooth simulator.

The purpose is to use Silabs to simulate device `/dev/ttyACM0`

**(I haven't done this part)**

## Regroup for A different tack

1.  Look at the Dockerfile and DockerCompose.yaml files and figure out how to build individual docker images.

e.g.
`sudo docker build -t markn/tiedie-ap:latest .` 

2.  Make sure you end up with the following docker images
```
markn@markn-Lenovo-G50-45:~/devel/TIEDIE/tiedie/gateway$ sudo docker images
[sudo] password for markn: 

REPOSITORY                  TAG       IMAGE ID       CREATED         SIZE
tiedie-ap                   dev       7a7942796a0d   19 hours ago    423MB
markn/tiedie-ap             latest    972b1343de04   20 hours ago    259MB
markn/domoticz              latest    dfef06851d6c   20 hours ago    230MB
adminer                     latest    b108b9a25fa6   7 days ago      250MB
iegomez/mosquitto-go-auth   latest    377a70d9c469   11 months ago   156MB
postgres                    13.1      407cece1abff   3 years ago     314MB
```

**Note**: the `markn/tiedie-ap:latest` is the docker image for tiedie gateway.
Once you have all the necessary images, you can start the command `docker compose up mosquito postgres`

3. Bring up the mosquitto and postgres containers:

```
markn@markn-Lenovo-G50-45:~/devel/TIEDIE/tiedie/gateway$ sudo docker compose up mosquitto postgres
[sudo] password for markn: 

WARN[0000] /home/markn/devel/TIEDIE/tiedie/gateway/docker-compose.yml: `version` is obsolete 
[+] Running 2/0
 ✔ Container gateway-postgres-1   Running                                                                                           0.0s 
 ✔ Container gateway-mosquitto-1  Running                                                                                           0.0s 
Attaching to mosquitto-1, postgres-1
mosquitto-1  | 1713413182: Saving in-memory database to /var/lib/mosquitto/mosquitto.db.

```

4. Run the Python application:

`python3 app.py --device mock`

```
venv) markn@markn-Lenovo-G50-45:~/devel/TIEDIE/tiedie/gateway$ python3 app.py --device /dev/ttyACM0
usage: app.py [-h] [-c [CPC]] [--cpc_lib_path CPC_LIB_PATH] [--cpc_tracing] [-l {CRITICAL,FATAL,ERROR,WARN,WARNING,INFO,DEBUG,NOTSET}]
              [--robust] [--no_crc] [--device {silabs,mock}]
              [conn]
app.py: error: argument --device: invalid choice: '/dev/ttyACM0' (choose from 'silabs', 'mock')
(venv) markn@markn-Lenovo-G50-45:~/devel/TIEDIE/tiedie/gateway$ python3 app.py --device silabs
usage: app.py [-h] [-c [CPC]] [--cpc_lib_path CPC_LIB_PATH] [--cpc_tracing] [-l {CRITICAL,FATAL,ERROR,WARN,WARNING,INFO,DEBUG,NOTSET}]
              [--robust] [--no_crc] [--device {silabs,mock}]
              [conn]
app.py: error: No serial device found. Please specify connection explicitly.
(venv) markn@markn-Lenovo-G50-45:~/devel/TIEDIE/tiedie/gateway$ python3 app.py --device mock
2024-04-17 21:32:51,115: INFO - System booted
 * Serving Flask app 'app_factory'
 * Debug mode: off
2024-04-17 21:32:51,164: INFO - WARNING: This is a development server. Do not use it in a production deployment. Use a production WSGI server instead.
 * Running on all addresses (0.0.0.0)
 * Running on https://127.0.0.1:8081
 * Running on https://192.168.0.25:8081
2024-04-17 21:32:51,164: INFO - Press CTRL+C to quit
```

5.  Proceed to the SDK client to try step 'Connect device'. Section https://github.com/aesclever/tiedie/blob/main/gateway/README.md#generate-api-keys

Or [connect with python sample app](https://github.com/aesclever/tiedie/blob/2bf9d358052f53834ea508c7993f3d00c3784c66/python-sdk/sample-python-app/README.md)

Refer back to the workfow tracking to steps in tiedie/README.md
