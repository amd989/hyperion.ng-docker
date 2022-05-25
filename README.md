![Hyperion](https://github.com/hyperion-project/hyperion.ng/blob/master/doc/logo_dark.png?raw=true#gh-dark-mode-only)
![Hyperion](https://github.com/hyperion-project/hyperion.ng/blob/master/doc/logo_light.png?raw=true#gh-light-mode-only)

This is a simple repository containining some `docker-compose.yml` sample files useful for running [hyperiong.ng](https://github.com/hyperion-project/hyperion.ng/#readme) as a service inside a docker environment, without having to rely on one of the closed sources docker images running wild out there. 
Just copy-paste and `docker-compose up -d` it.

The docker-compose file is quite simple:

1. It's based on the [official Debian 11 (bullseye) docker image](https://hub.docker.com/_/debian)
2. It downloads the hyperion official package from the [official hyperion apt package repository](https://apt.hyperion-project.org/)
3. Maps the `/config` dirctory as an external volume, to keep your settings
4. Runs hyperiond service as non-root user. Default UID:GID are 1000:1000 but they can be easily changed adding a `.env` file

The setup is done on first run. 

Sadly, the resulting image is not exaclty slim at ~500MB, because hyperion has lots of dependencies. Since many of them are for the Desktop/Qt UI, it should be possible to slim the image up by cherry picking the ones not used but the cli service, but that's probably not really worth it.

On the other hand, the running service does not need lots of RAM (on my system takes ~64MB without the cache).

### Default configuration

In this configuration all the main hyperion ports are mapped on your docker host. You can reach the web ui going either to http://youdockerhost:8090 or https://youdockerhost:8092

```yaml
version: '3.3'

services:
  hyperionng:
    image: debian:bullseye
    container_name: hyperionng
    command: bash -c "addgroup -q --gid ${UID:-1100} hyperion &&
                    adduser -q --uid ${UID:-1100} --gid ${GID:-1100} --disabled-password --no-create-home hyperion &&
                    apt-get update &&
                    apt-get install -y wget gpg sudo &&
                    wget -qO /tmp/hyperion.pub.key https://apt.hyperion-project.org/hyperion.pub.key &&
                    gpg --dearmor -o - /tmp/hyperion.pub.key > /usr/share/keyrings/hyperion.pub.gpg &&
                    echo \"deb [signed-by=/usr/share/keyrings/hyperion.pub.gpg] https://apt.hyperion-project.org/ bullseye main\" > /etc/apt/sources.list.d/hyperion.list &&
                    apt-get update &&
                    apt-get install -y hyperion &&
                    apt-get clean &&
                    sudo -u hyperion /usr/bin/hyperiond -v --service -u /config"
    ports:
      - "19400:19400"
      - "19444:19444"
      - "19445:19445"
      - "8090:8090"
      - "8092:8092"
    volumes:
      - hyperiong-config:/config
    restart: unless-stopped
    volumes:
      - hyperionng-data
```

If you want to use different UID and GID, you can add a `.env` file in the same folder of your `docker-compose.yml` file:

```properties
UID=1100
GID=1100
```