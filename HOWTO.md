
# Reverse Proxy in the IoT Network
In an IoT gateway network, a reverse proxy can be strategically placed to manage and optimize incoming and outgoing traffic between external clients and internal IoT devices or services. 

**Here is a simplified diagram illustrating the placement of a reverse proxy**:

            External Clients
                 |
                 |
            [Reverse Proxy]
                 |
                 |    IoT Gateway Network
                 |    +------------------+
                 |    |     IoT Devices  |
                 |    +------------------+
                 |    |     Services     |
                 |    +------------------+
                 |
           Internal IoT Devices

By placing the reverse proxy in front of the IoT gateway network, you can centralize control over incoming and outgoing traffic, improve security by implementing access controls and filtering, enhance performance through caching and load balancing, and simplify management of IoT services and devices.

**Where**:

- **External Clients**: These could be users accessing IoT services or devices over the internet.

- **Reverse Proxy**: Acts as a gateway between external clients and the internal IoT gateway network. It intercepts requests from clients and forwards them to the appropriate IoT devices or services within the network. It can also perform functions like load balancing, caching, SSL termination, and security filtering.

- **IoT Gateway Network**: Represents the internal network where IoT devices and services reside. The reverse proxy serves as a protective barrier for this network, shielding internal resources from direct exposure to the internet and providing an additional layer of security.
    


# How to scalfold the tiedie gateway



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

### Prep Knowledge on How to start the Gateway

**Note1**:  There is a hint in "On a linux host, *where usb passthrough is supported in docker*"

You can use the --device flag that use can use to access USB devices without --privileged mode:

`docker run -t -i --device=/dev/ttyUSB0 ubuntu bash`

Alternatively, assuming your USB device is available with drivers working, etc. on the host in /dev/bus/usb, you can mount this in the container using privileged mode and the volumes option. For example:

`docker run -t -i --privileged -v /dev/bus/usb:/dev/bus/usb ubuntu bash`

3.  Run the docker compose command to start the tiedie gateway container

*In `Docker-compose.yml` we can have volume: ' - /dev/bus/usb:/dev/bus/usb'*


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
markn@markn-Lenovo-G50-45:~/devel/TIEDIE/tiedie/gateway$ sudo docker compose -f docker-compose.debug.yml up mosquitto postgres
WARN[0000] /home/markn/devel/TIEDIE/tiedie/gateway/docker-compose.debug.yml: `version` is obsolete 
WARN[0000] Found orphan containers ([gateway-adminer-1]) for this project. If you removed or renamed this service in your compose file, you can run this command with the --remove-orphans flag to clean it up. 
[+] Running 0/2
 ⠦ Container gateway-mosquitto-1  Creat...                                 2.5s 
 ⠦ Container gateway-postgres-1   Create...                                2.5s 
Attaching to mosquitto-1, postgres-1
postgres-1   | The files belonging to this database system will be owned by user "postgres".
postgres-1   | This user must also own the server process.
postgres-1   | 
postgres-1   | The database cluster will be initialized with locale "en_US.utf8".
postgres-1   | The default database encoding has accordingly been set to "UTF8".
postgres-1   | The default text search configuration will be set to "english".
postgres-1   | 
postgres-1   | Data page checksums are disabled.
postgres-1   | 
postgres-1   | fixing permissions on existing directory /var/lib/postgresql/data ... ok
postgres-1   | creating subdirectories ... ok
postgres-1   | selecting dynamic shared memory implementation ... posix
mosquitto-1  | 1714125356: Error: Unable to open include_dir '/etc/mosquitto/conf.d'.
mosquitto-1  | 1714125356: Error found at /mosquitto/config/mosquitto.conf:21.
postgres-1   | selecting default max_connections ... 100
postgres-1   | selecting default shared_buffers ... 128MB
postgres-1   | selecting default time zone ... Etc/UTC
postgres-1   | creating configuration files ... ok
postgres-1   | running bootstrap script ... ok
mosquitto-1 exited with code 3
postgres-1   | performing post-bootstrap initialization ... ok
postgres-1   | syncing data to disk ... ok
postgres-1   | 
postgres-1   | 
postgres-1   | Success. You can now start the database server using:
postgres-1   | 
postgres-1   |     pg_ctl -D /var/lib/postgresql/data -l logfile start
postgres-1   | 
postgres-1   | initdb: warning: enabling "trust" authentication for local connections
postgres-1   | You can change this by editing pg_hba.conf or using the option -A, or
postgres-1   | --auth-local and --auth-host, the next time you run initdb.
postgres-1   | waiting for server to start....2024-04-26 09:56:01.527 UTC [46] LOG:  starting PostgreSQL 13.1 (Debian 13.1-1.pgdg100+1) on x86_64-pc-linux-gnu, compiled by gcc (Debian 8.3.0-6) 8.3.0, 64-bit
postgres-1   | 2024-04-26 09:56:01.570 UTC [46] LOG:  listening on Unix socket "/var/run/postgresql/.s.PGSQL.5432"
postgres-1   | 2024-04-26 09:56:01.707 UTC [47] LOG:  database system was shut down at 2024-04-26 09:56:00 UTC
postgres-1   | 2024-04-26 09:56:01.760 UTC [46] LOG:  database system is ready to accept connections
postgres-1   |  done
postgres-1   | server started
postgres-1   | CREATE DATABASE
postgres-1   | 
postgres-1   | 
postgres-1   | /usr/local/bin/docker-entrypoint.sh: running /docker-entrypoint-initdb.d/init.sql
postgres-1   | CREATE DATABASE
postgres-1   | 
postgres-1   | 
postgres-1   | 2024-04-26 09:56:04.313 UTC [46] LOG:  received fast shutdown request
postgres-1   | waiting for server to shut down....2024-04-26 09:56:04.353 UTC [46] LOG:  aborting any active transactions
postgres-1   | 2024-04-26 09:56:04.359 UTC [46] LOG:  background worker "logical replication launcher" (PID 53) exited with exit code 1
postgres-1   | 2024-04-26 09:56:04.362 UTC [48] LOG:  shutting down
postgres-1   | 2024-04-26 09:56:04.598 UTC [46] LOG:  database system is shut down
postgres-1   |  done
postgres-1   | server stopped
postgres-1   | 
postgres-1   | PostgreSQL init process complete; ready for start up.
postgres-1   | 
postgres-1   | 2024-04-26 09:56:04.733 UTC [1] LOG:  starting PostgreSQL 13.1 (Debian 13.1-1.pgdg100+1) on x86_64-pc-linux-gnu, compiled by gcc (Debian 8.3.0-6) 8.3.0, 64-bit
postgres-1   | 2024-04-26 09:56:04.734 UTC [1] LOG:  listening on IPv4 address "0.0.0.0", port 5432
postgres-1   | 2024-04-26 09:56:04.734 UTC [1] LOG:  listening on IPv6 address "::", port 5432
postgres-1   | 2024-04-26 09:56:04.801 UTC [1] LOG:  listening on Unix socket "/var/run/postgresql/.s.PGSQL.5432"
postgres-1   | 2024-04-26 09:56:04.881 UTC [83] LOG:  database system was shut down at 2024-04-26 09:56:04 UTC
postgres-1   | 2024-04-26 09:56:04.945 UTC [1] LOG:  database system is ready to accept connections

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
