# Docker Gitlab CE/Gitlab Runner/LAVA project

This project aim to provide an easy to install CI env using Docker containers for Embedded devices.

Gitlab CE and Runner are used to manage the sources, team work and CI pipeline. It is a all in one tool that provides issue managment through tickets and code review with merge request. Teams can be managed with the group tab and the dashboard. The CI can be scheduled.
For example, you can run a build when a commit is accepted on the master branch so you are sure it does not break the source.

LAVA is an advanced tool provided by Linaro which helps testing the generated images or binaries on devices and return status to validate or not the build. It can be connected to the Gitlab CI pipeline. For example, when creating a release, you can run a complete pipeline which validate the compilation and on board testing.

The project use Postgres for the database and Redis for the key managment.

## Install dependencies
Docker:
```
sudo apt-get remove docker docker-engine docker.io
sudo apt-get update
sudo apt-get install apt-transport-https ca-certificates curl software-properties-common
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo apt-key add -
sudo add-apt-repository "deb [arch=amd64] https://download.docker.com/linux/ubuntu $(lsb_release -cs) stable"
sudo apt-get update
sudo apt-get install docker-ce
```
Docker-compose:
```
sudo curl -L "https://github.com/docker/compose/releases/download/1.23.1/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose
sudo chmod +x /usr/local/bin/docker-compose
```

Use Docker without sudo:
```
sudo groupadd docker
sudo usermod -aG docker $USER
su - $USER
```
Then logout and login.

## Launch the Docker CI
Run those steps:
```
source .env
docker-compose up -d
```

Note: By default Gitlab is running on localhost on the port 1000.

To change any parameters, version or other, edit those files: .env, docker-compose.yml

## Setup Gitlab CE and Gitlab Runner

### Create a Gitlab root account
The Gitlab CE web interface is accessible by default at: http://localhost:1000

The first time you launch Gitlab, it will ask for a new root password. Follow the step.

### Register a Gitlab Runner
A Gitlab Runner is required if you plan to use the CI pipeline provide by Gitlab CE.

The Runner support several execution modes in the pipeline like VM, shell, Docker, ... Check the Gitlab Runner documentation for further informations.

CONTAINER_NAME_OR_ID: {gitlab-runner1, gitlab-runner2, ...}
TOKEN: From Giltab CE Runner parameters (Shared Runner: http://localhost:1000/admin/runners OR Project Runner: http://localhost:1000/<USER>/<PROJECT>/settings/ci_cd)
HOSTNANE: gitlab
EXECUTOR: docker, shell, VM, ...
PROJECT_DOCKER_NETWORK: example: docker_gitlab_lava_default

Gitlab Runner registrer example using a Docker build environment:
```
docker exec -it <CONTAINER_NAME_OR_ID> gitlab-runner register -n -r <TOKEN> -u http://<HOSTNAME> \
--executor <EXECUTOR> \
--docker-image <DOCKER_IMAGE_FOR_COMPILATION> \
--docker-network-mode <PROJECT_DOCKER_NETWORK> \
--docker-volumes "/var/run/docker.sock:/var/run/docker.sock" \
--docker-volumes <LIST_OF_VOLUME_REQUIRED_FOR_THE_COMPILATION>
```

Note:
    -Never forget this command is using a Docker build environment
    ```
    --docker-volumes "/var/run/docker.sock:/var/run/docker.sock"
    ```
    It prevent the Docker-in-Docker situation, which can cause memory corruption.

    - If the Docker image used inside the Runner is local. You can prevent the pull from the Docker Hub and avoid a failure by giving the following option:
    ```
    --docker-pull_policy "if-not-present"
    ```

    - To go further in the Gitlab Runner configuration, check the official documentation to know how to edit the file config.toml in "./volumes/gitlab-runner<N>/"
    https://docs.gitlab.com/runner/configuration/advanced-configuration.html

### Manage the CI pipeline
The Gitlab pipeline is easy to setup. You can create jobs in the pipeline by using a configuration file.
First create a ".gitlab-ci.yml" at the root of your project.

Here is an example of a basic pipeline you can create to {build, test, deploy} on the "master" branch:
```
stages:
  - build
  - test
  - deploy

only:
  - master

job build:
  stage: build
  script:
    - <COMMANDS>

job test:
  stage: test
  script:
    - <COMMANDS>

job deploy:
  stage: deploy
  script:
    - <COMMANDS>
```

More details here: https://docs.gitlab.com/ee/ci/yaml/

## Setup LAVA
NOTES: TO COMPLETE
https://lists.lavasoftware.org/pipermail/lava-users/2018-December/001441.html
https://github.com/danrue/lava.home.therub.org

<!-- Setup a network:
```
docker network create --subnet=172.0.1.0/24 dockernet
```
### LAVA Dispatcher to manage the devices
```
docker pull lavasoftware/lava-dispatcher:2018.10
docker run --net dockernet --ip 172.0.1.4 -it lavasoftware/lava-dispatcher:2018.10
```
### LAVA Server
```
docker pull lavasoftware/lava-server:2018.10
docker run --net dockernet --ip 172.0.1.5 -it lavasoftware/lava-server:2018.10
docker exec -it <CONTAINER_ID> lava-server manage users add --staff --superuser --email <EMAIL> --passwd <PASSWORD> <USERNAME>
```

## Running LAVA Server
```
docker run --net dockernet --ip 172.0.1.5 -it lavasoftware/lava-server:2018.10
```


docker exec -it lavasoftware/lava-server:2018.10 lava-server manage users add --staff --superuser --email "" --passwd root root
 -->
