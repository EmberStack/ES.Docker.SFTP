# SFTP ([SSH File Transfer Protocol](https://en.wikipedia.org/wiki/SSH_File_Transfer_Protocol)) server using [OpenSSH](https://en.wikipedia.org/wiki/OpenSSH)
This project provides a Docker image for hosting a SFTP server. Included are `Docker` (`docker-cli` and `docker-compose`) and `Kubernetes` (`kubectl` and `helm`) deployment scripts

[![Build Status](https://dev.azure.com/emberstack/OpenSource/_apis/build/status/docker-sftp?branchName=master)](https://dev.azure.com/emberstack/OpenSource/_build/latest?definitionId=16&branchName=master)
[![Release](https://img.shields.io/github/release/emberstack/docker-sftp.svg?style=flat-square)](https://github.com/emberstack/docker-sftp/releases/latest)
[![GitHub Tag](https://img.shields.io/github/tag/emberstack/docker-sftp.svg?style=flat-square)](https://github.com/emberstack/docker-sftp/releases/latest)
[![Docker Image](https://images.microbadger.com/badges/image/emberstack/sftp.svg)](https://microbadger.com/images/emberstack/sftp)
[![Docker Version](https://images.microbadger.com/badges/version/emberstack/sftp.svg)](https://microbadger.com/images/emberstack/sftp)
[![Docker Pulls](https://img.shields.io/docker/pulls/emberstack/sftp.svg?style=flat-square)](https://hub.docker.com/r/emberstack/sftp)
[![Docker Stars](https://img.shields.io/docker/stars/emberstack/sftp.svg?style=flat-square)](https://hub.docker.com/r/remberstack/sftp)
[![license](https://img.shields.io/github/license/emberstack/docker-sftp.svg?style=flat-square)](LICENSE)


> Supports architectures: `amd64`. Coming soon: `arm` and `arm64`

## Usage

The SFTP server can be easily deployed to any platform that can host containers based on Docker.
Below are deployment methods for:
- Docker CLI
- Docker-Compose
- Kubernetes using Helm (recommended for Kubernetes)
- Kubernetes (manual)

Process:
1) Create server configuration
2) Mount volumes as needed
3) Set host file for consistent server fingerprint

### Configuration

The SFTP server uses a `json` based configuration file for default server options and to define users. This file has to be mounted on `/sftp/config/sftp.json` inside the container.
Environment variable based configuration is not supported (see the `Advanced Configuration` section below for the reasons).

Below is the simplest configuration file for the SFTP server:

```json
{
    "global": {
        "chroot": {
            "directory": "%h",
            "startPath": "sftp"
        },
        "directories": [ "sftp" ]
    },
    "users": [
        {
            "username": "demo",
            "password": "password"
        },
        {
            "username": "demo2",
            "password": "password"
        }
    ]
}
```
This configuration creates a user `demo` with the password `demo`. 
A directory "sftp" is created for each user in the own home and is accessible for read/write. 
The user is `chrooted` to the `/home/demo` directory. Upon connect, the start directory is `sftp`.

You can add additional users, default directories or customize start directories per user. You can also define the `uid` and `gid` for each user. See the `Advanced Configuration` section below for all configuration options.


### Deployment using Docker CLI

> Simple Docker CLI run

```shellsession
$ docker run -p 22:22 -d emberstack/sftp --name sftp
```
This will start a SFTP in the container `sftp` with the default configuration. You can connect to it and login with the `user: demo` and `password: demo`.

> Provide your configuration

```shellsession
$ docker run -p 22:22 -d emberstack/sftp --name sftp -v /host/sftp.conf:/sftp/config/sftp.conf:ro
```
This will override the default (`/sftp/config/sftp.conf`) configuration with the one from the host `/host/sftp.conf`.

> Mount a directory from the host for the user 'demo'

```shellsession
$ docker run -p 22:22 -d emberstack/sftp --name sftp -v /host/sftp.conf:/sftp/config/sftp.conf:ro -v /host/demo:/home/demo/sftp
```
This will mount the `demo` directory from the host on the `sftp` directory for the "demo" user.


### Deployment using Docker Compose

> Simple docker-compose configuration

Create a docker-compose configuration file:
```yaml
version: '3'
services:
  sftp:
    image: "emberstack/sftp"
    ports:
      - "22:22"
    volumes:
      - ../config-samples/sample.sftp.json:/sftp/config/sftp.json:ro
```
And run it using docker-compose
```shellsession
$ docker-compose -p sftp -f docker-compose.yaml up -d
```

The above configuration is available in the `deploy\docker-compose` folder in this repository. You can use it to start customizing the deployment for your environment.



### Deployment to Kubernetes using Helm

Use Helm to install the latest released chart:
```shellsession
$ helm repo add emberstack https://emberstack.github.io/helm-charts
$ helm repo update
$ helm upgrade --install sftp emberstack/sftp
```

You can customize the values of the helm deployment by using the following Values:

| Parameter                                                   | Description                                                                      | Default                                                 |
| ------------------------------------                        | -------------------------------------------------------------------------------- | ------------------------------------------------------- |
| `nameOverride`                                              | Overrides release name                                                           | `""`                                                    |
| `fullnameOverride`                                          | Overrides release fullname                                                       | `""`                                                    |
| `image.repository`                                          | Container image repository                                                       | `emberstack/sftp`                                       |
| `image.tag`                                                 | Container image tag                                                              | `latest`                                                |
| `image.pullPolicy`                                          | Container image pull policy                                                      | `Always` if `image.tag` is `latest`, else `IfNotPresent`|
| `storage.volumes`                                           | Defines additional volumes for the pod                                           | `{}`                                                    |
| `storage.volumeMounts`                                      | Defines additional volumes mounts for the sftp container                         | `{}`                                                    |
| `configuration.global.chroot.directory`                     | Global chroot directory for the `sftp` user group. Can be overriden per-user     | `"%h"`                                                  |
| `configuration.global.chroot.startPath`                     | Start path for the `sftp` user group. Can be overriden per-user                  | `"sftp"`                                                |
| `configuration.global.directories`                          | Directories that get created for all `sftp` users. Can be appended per user      | `["sftp"]`                                              |
| `configuration.users`                                       | Array of users and their properties                                              | Contains `demo` user by default                         |
| `configuration.users[].username`                            | Set the user's username                                                          | N/A                                                     |
| `configuration.users[].password`                            | Set the user's password. If empty or `null`, password authentication is disabled | N/A                                                     |
| `configuration.users[].passwordEncrypted`                   | `true` or `false`. Indicates if the password value is already encrypted          | `false`                                                 |
| `configuration.users[].passwordEncrypted`                   | `true` or `false`. Indicates if the password value is already encrypted          | `false`                                                 |
| `configuration.users[].chroot`                              | If set, will override global `chroot` settings for this user.                    | `null`                                                  |
| `configuration.users[].directories`                         | Array of additional directories created for this user                            | `null`                                                  |
| `initContainers`                                            | Additional initContainers for the pod                                            | `{}`                                                    |
| `resources`                                                 | Resource limits                                                                  | `{}`                                                    |
| `nodeSelector`                                              | Node labels for pod assignment                                                   | `{}`                                                    |
| `tolerations`                                               | Toleration labels for pod assignment                                             | `[]`                                                    |
| `affinity`                                                  | Node affinity for pod assignment                                                 | `{}`                                                    |

> Find us on [Helm Hub](https://hub.helm.sh/charts/emberstack)


### Deployment to Kubernetes using kubectl
Each release (found on the [Releases](https://github.com/EmberStack/docker-sftp/releases) GitHub page) contains the manual deployment file (`sftp.yaml`).

```shellsession
$ kubectl apply -f https://github.com/EmberStack/docker-sftp/releases/latest/download/sftp.yaml
```

## Advanced Configuration

TODO: This section is under development due to the number of configuration options being added. Please open an issue on the [emberstack/docker-sftp](https://github.com/emberstack/docker-sftp) project if you need help.



## Final Word
This project is a work in progress. More features and configuration options will be added. If you want to contribute or have feedback, please feel free to create an Issue or contrinute with a Pull Request.
This project was initially inspired by [atmoz/sftp](https://github.com/atmoz/sftp) but has changed from the original design to increase flexibility and range of supported deployment options. This is no longer a lightweight SFTP server, the target being a rich set of features, to the detriment of simplicity. If you're looking for a simple SFTP docker image, please consider the [atmoz/sftp](https://github.com/atmoz/sftp) project.