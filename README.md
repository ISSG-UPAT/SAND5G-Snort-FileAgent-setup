# Snort Docker Image

## Description of the Project

This repository is part of the SAND5G project, which aims to enhance security in 5G networks. FileAgent is a tool designed to facilitate the management of Snort rules in a containerized environment.

5G -and beyond- networks provide a strong foundation for EU’s digital transformation and are becoming one of the Union’s key assets to compete in the global market.

Securing 5G networks and the services running on top of them requires high quality technical security solutions and also strong collaboration at the operational level.

https://sand5g-project.eu

<img src="https://sand5g-project.eu/wp-content/uploads/2024/06/SAND5G-logo-600x137.png" alt="SAND5G" width="300" height="68">

## What it is

This is the docker-compose configuration that utilizes Snort in Docker and the Fileagent. Find more information about the docker images in the following links

- [Snort-Docker-SAND5G](https://github.com/ISSG-UPAT/Snort-Docker-SAND5G)
- [FileAgent-Docker-SAND5G](https://github.com/ISSG-UPAT/FileAgent-Docker-SAND5G)

## How to use

### Download the docker-compose file

```bash
curl https://raw.githubusercontent.com/ISSG-UPAT/SAND5G-Snort-FileAgent-setup/refs/heads/main/docker-compose.yml -o docker-compose.yml
```

### Commands for the docker-compose

```bash
docker compose up -d
```

To view logs run

```bash
docker compose logs -f
```

To stop the containers run

```bash
docker compose down
```

### Docker compose file :

```yaml
services:
  snort3:
    image: issgupat/snort-docker-sand5g:latest
    hostname: snort3
    network_mode: "host"
    cap_add:
      - NET_ADMIN
      - NET_RAW
    volumes:
      - custom_data:/home/snorty/custom
      - alerts_data:/home/snorty/alerts
      - /etc/localtime:/etc/localtime:ro
    environment:
      - TZ=Europe/Athens
      - RULES_FILE=/home/snorty/custom/tls.rules
      - INTERFACE=ens3
    privileged: true
    stdin_open: true
    tty: true
    restart: unless-stopped

  fileagent:
    image: issgupat/fileagent-docker-sand5g:latest
    environment:
      - PORT=8000
      - HOST="0.0.0.0"
      - FILE=tls.rules
      - DIRECTORY=/app/custom
    hostname: fileagent
    network_mode: "host"
    ports:
      - "8000:8000"
    volumes:
      - custom_data:/app/custom

volumes:
  alerts_data:
    driver: local
    driver_opts:
      type: none
      o: bind
      device: ./volumes/alerts

  custom_data:
    driver: local
    driver_opts:
      type: none
      o: bind
      device: ./volumes/custom
```

### Variables of the Dockers

#### Snort

| Variable        | Required | Default Value                        | Description                                            |
| --------------- | -------- | ------------------------------------ | ------------------------------------------------------ |
| RULES_FILE      | OPTIONAL | /home/snorty/custom/local.rules      | Which rule file to use                                 |
| SNORT_CONF_FILE | OPTIONAL | /home/snorty/custom/custom_snort.lua | Which configuration file to use                        |
| SNORT_ALERTS    | YES      | /home/snorty/alerts                  | Which folder to use for alert output                   |
| TZ              | YES      | Europe/Athens                        | Used to have accurate timestamps                       |
| INTERFACE       | YES      | <>                                   | The interface to monitor.                              |
|                 |          |                                      | Default is the first interface available in the system |

#### FileAgent

| Variable  | Required | Default Values | Description                                  |
| --------- | -------- | -------------- | -------------------------------------------- |
| PORT      | OPTIONAL | 8000           | The port on which the FileAgent will run.    |
| HOST      | OPTIONAL | "0.0.0.0"      | The host on which the FileAgent will run.    |
| DIRECTORY | OPTIONAL | <>             | The directory to be monitored.               |
|           |          |                | Defaults to the parent directory of the file |
| FILE      | YES      |                | The file to be monitored.                    |

## License

This project is licensed under the MIT License. See the [LICENSE](LICENSE) file for details.
