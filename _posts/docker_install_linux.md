---
layout: post
title: Docker installation for Linux
---

## Docker installation for Linux

Docker can be installed in three ways

- Script: https://get.docker.com  (recommended way)
- Store: Go to docker store and install as per instruction
- docker-machine: https://github.com/docker/machine/releases

There two distribution version CE and EE.

- EE (Enterprise Edition): is more stable and has longer time support (6 month approx). This is paid version of docker

- CE (Community Edition): is beta version and changes more often (Once a month approx). This is free version of docker.

### Procedure for script method

- Run the script  download from https://get.docker.com

- Adding user to docker group

  ```bash
  sudo usermod -aG docker <username>
  ```

  :bell: That adding user can work in most OS (ubuntu), but in RedHat it might not work. So we need to use root in that case.

  

- Test if user is add 

  ```
  docker version
  ```

  :bell:If this command gives error, perhaps user is not add or this feature is not supported by OS.

- Now once docker is installed, install docker-compose and docker-machine using below link

  - https://docs.docker.com/machine/install-machine/
  - https://docs.docker.com/compose/install/



​	:bell: Bells and whistles

- For **zsh** feature refer this link  https://www.bretfisher.com/shell/

### Login and Logout docker

:octopus: login into docker

```bash
docker login
```

- docker login stores authentication key in profile of the user
- If we log out this stored authentication file is deleted

:octopus: logout docker

```bash
docker logout
```


