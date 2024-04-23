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

**I found nothing useful on Silabs!!!**

**Possibly a better reference**: https://www.baeldung.com/linux/make-virtual-serial-port


## Regroup for A Different Tack

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

> Maintain two consoles:

- Console#1 `docker compose up mosquito postgress`

```
markn@markn-Lenovo-G50-45:~/devel/TIEDIE/tiedie/gateway$ sudo docker compose up mosquitto postgres
[sudo] password for markn: 

WARN[0000] /home/markn/devel/TIEDIE/tiedie/gateway/docker-compose.yml: `version` is obsolete 
[+] Running 2/0
 ✔ Container gateway-postgres-1   Running                                                                                           0.0s 
 ✔ Container gateway-mosquitto-1  Running                                                                                           0.0s 
Attaching to mosquitto-1, postgres-1
mosquitto-1  | 1713413182: Saving in-memory database to /var/lib/mosquitto/mosquitto.db.
mosquitto-1  | 1713413989: Client auto-0434420C-9B9D-A19D-C283-2A4483B313FC closed its connection.
mosquitto-1  | 1713414725: New connection from 172.18.0.1:40236 on port 8883.
mosquitto-1  | 1713414725: New client connected from 172.18.0.1:40236 as auto-F99F8616-2138-F9AC-C399-C0CA51B2E5E0 (p2, c1, k60, u'admin').

...


mosquitto-1  | 1713902838: Client auto-F3874432-BBDE-D909-4BDA-87F5EF0A707A closed its connection.
mosquitto-1  | 1713902843: New connection from 172.18.0.1:34036 on port 8883.
mosquitto-1  | 1713902843: New client connected from 172.18.0.1:34036 as auto-783EDEF3-474F-022E-2A60-5E1FB36F5CD4 (p2, c1, k60, u'admin').
mosquitto-1  | 1713903213: Saving in-memory database to /var/lib/mosquitto/mosquitto.db.
mosquitto-1  | 1713903615: Client auto-783EDEF3-474F-022E-2A60-5E1FB36F5CD4 closed its connection.

```

4. Run the Python application:

- Console#2
  ```
  ifconfig -a

  python3 app.py -h
  ```

Python App syntax:
```
usage: app.py [-h] [-c [CPC]] [--cpc_lib_path CPC_LIB_PATH] [--cpc_tracing] [-l {CRITICAL,FATAL,ERROR,WARN,WARNING,INFO,DEBUG,NOTSET}]
              [--robust] [--no_crc] [--device {silabs,mock}]
              [conn]

positional arguments:
  conn                  Serial or TCP connection parameter. See the examples for details. (default: None)

options:
  -h, --help            show this help message and exit
  -c [CPC], --cpc [CPC]
                        CPC instance (default: None)
  --cpc_lib_path CPC_LIB_PATH
                        CPC shared library path (default: /usr/local/lib/libcpc.so)
  --cpc_tracing         Enable CPC tracing (default: False)
  -l {CRITICAL,FATAL,ERROR,WARN,WARNING,INFO,DEBUG,NOTSET}, --log {CRITICAL,FATAL,ERROR,WARN,WARNING,INFO,DEBUG,NOTSET}
                        Log level (default: INFO)
  --robust              Enable robust communication (default: False)
  --no_crc              Disable CRC checking for robust communication. Ignored if robust communication is disabled. (default: True)
  --device {silabs,mock}
                        Device (default: silabs)

examples:
  app.py                 Try to autodetect serial port
  app.py COM4            Open serial port on Windows
  app.py /dev/ttyACM0    Open serial port on POSIX
  app.py 192.168.1.10    Open TCP port
  app.py -c              Open default CPC daemon instance
  app.py -c cpcd_1       Open CPC daemon instance
```

**The python client app**:

- Modify file `app.py` to connect via IPv6 (AF_INET) address.

```
...

    app.run(host="::", port=8081, ssl_context=context,
            request_handler=PeerCertWSGIRequestHandler)

...
```

**Connect to a mocked device via MAC address**:

`python3 app.py -lINFO --robust  --device mock 02:42:fd:eb:17:2c`

**Output**:

```
2024-04-23 13:07:23,564: INFO - System booted
 * Serving Flask app 'app_factory'
 * Debug mode: off
2024-04-23 13:07:23,622: INFO - WARNING: This is a development server. Do not use it in a production deployment. Use a production WSGI server instead.
 * Running on all addresses (::)
 * Running on https://[::1]:8081
 * Running on https://[2601:641:380:75e0::3]:8081
2024-04-23 13:07:23,622: INFO - Press CTRL+C to quit

```


**What's next?**

Refer back to the workfow tracking to steps in tiedie/README.md
