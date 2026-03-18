# Installing OpenBOS

Installing openBOS consists of building the docker images that collectively make up the kernel and the drivers you want to use.

## Prework
To install openBOS you'll need docker. If you don't have it install it. Follow the instruction [here](https://docs.docker.com/engine/install/).

Once installed, grant docker access to the Docker socket.
``` bash
sudo usermod -aG docker $USER
```
You MUST log out and log back in on Linux for this to take effect.


## Cloning the repo
Presently, the repo is private. To access it, make sure your public ssh key has been added to your github profile so this operation works. For a refresher review [this](https://docs.github.com/en/authentication/connecting-to-github-with-ssh/adding-a-new-ssh-key-to-your-github-account) page.

Start by cloning the repo:
``` bash
git clone --recursive git@github.com:jamesryancoleman/bos.git bos
```

This project is designed to be run rootless. So you'll need to update your docker host environment variable to reflect that. 

```bash
cd ~
echo 'export DOCKER_HOST=unix:///run/user/$(id -u)/docker.sock' >> ~/.bashrc && source ~/.bashrc
```

## Launching the system
When the repo is cloned launch the kernel services with:

```bash
docker compose up sysmod devctrl history orchestrator forecast events spog -d
```

You may get an error that part 80 is already in use on ubuntu. If so disable the apache service (pre-installed in Ubuntu) or re change the spog port in your docker-compose.yml file.

## BOPTEST Support
If you want to use openBOS with [BOPTEST](https://github.com/ibpsa/project1-boptest) follow the github instructions to install and launch the BOPTEST. For example:

```bash
git clone https://github.com/ibpsa/project1-boptest.git boptest 
cd boptest/
docker compose up web worker provision
```

Once docker is up. You'll need to launch the boptest driver

```bash
docker compose up boptest -d
```

the External Reference format for boptest points using the provided driver is: `boptest://bestest_air/zon_weaSta_reaWeaTDryBul_y`. Where the uri parts correspond to `boptest://[TEST_CASE]/[REST_API_POINT_NAME]`