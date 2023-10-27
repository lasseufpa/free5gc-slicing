# free5GC Slincing

This repository is a docker compose version of [free5GC](https://github.com/free5gc/free5gc) for stage 3, implementing network slicing at the core level. It's inspired by [free5gc-docker-compose](https://github.com/calee0219/free5gc-docker-compose)

To configure your own settings, please navigate to the [config](./config) folder and [docker-compose.yaml](docker-compose.yaml) files."
## Prerequisites

- Free5GC Release 3.3.0
- I use Ubuntu 20.04 LTS and Kernel 5.15.0
- [GTP5G kernel module](https://github.com/free5gc/gtp5g): needed to run the UPF
   
```bash
git clone https://github.com/free5gc/gtp5g.git && cd gtp5g
make clean && make
sudo make install
```

- [Docker Engine](https://docs.docker.com/engine/install): needed to run the Free5GC 

Run the following command to uninstall all conflicting packages:

```bash
for pkg in docker.io docker-doc docker-compose podman-docker containerd runc; do sudo apt-get remove $pkg; done
```

Set up Docker's Apt repository.

```bash
# Add Docker's official GPG key:
sudo apt-get update
```
```bash
sudo apt-get install ca-certificates curl gnupg
```

```bash
sudo install -m 0755 -d /etc/apt/keyrings

```

```bash
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /etc/apt/keyrings/docker.gpg
```

```bash
sudo chmod a+r /etc/apt/keyrings/docker.gpg
```

```bash
# Add the repository to Apt sources:
echo \
  "deb [arch="$(dpkg --print-architecture)" signed-by=/etc/apt/keyrings/docker.gpg] https://download.docker.com/linux/ubuntu \
  "$(. /etc/os-release && echo "$VERSION_CODENAME")" stable" | \
  sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
sudo apt-get update
```

Install
```bash
sudo apt-get install docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin
```
Verify docker installation
```bash
sudo docker run hello-world
```

## Start free5gc

Because we need to create tunnel interface, we need to use privileged container with root permission.


```bash
# Clone the project
git clone https://github.com/HeitorAnglada/free5gc-compose.git
cd free5gc-compose
```
# clone free5gc sources

Note:

Dangling images may be created during the build process. It is advised to remove them from time to time to free up disk space.

```bash
docker rmi $(docker images -f "dangling=true" -q)
```

### Run free5GC

You can create free5GC containers based on local images or docker hub images:

```bash
# use images from docker hub
docker compose up # add -d to run in background mode
```

Destroy the established container resource after testing:

```bash
# Remove established containers (remote images)
docker compose rm
```

## Troubleshooting

You can see logs for each service using `docker logs` command. For example, to access the logs of the *SMF* you can use:

```console
docker logs smf
```

Please refer to the [wiki](https://github.com/free5gc/free5gc/wiki) for more troubleshooting information.

## Integration with external gNB/UE simulators

The integration with the [UERANSIM](https://github.com/aligungr/UERANSIM) eNB/UE simulator is documented [here](https://www.free5gc.org/installations/stage-3-sim-install/). 

You can also refer to this [issue](https://github.com/free5gc/free5gc-compose/issues/26) to find out how you can configure the UPF to forward traffic between the [UERANSIM](https://github.com/aligungr/UERANSIM) to the DN (eg. internet) in a docker environment.

This [issue](https://github.com/free5gc/free5gc-compose/issues/28) provides detailed steps that might be useful.

## Scenario

<p align="center">
  <a href="https://github.com/HeitorAnglada/free5gc-compose">
    <img src="https://i.ibb.co/3Tf7mdc/free5gc-slice-cenario.png" width="1000" alt="free5gc-slice">
  </a>
</p>

We will test a core slicing scenario consisting of a gNB with two UEs in different slices connected to it. Each UE connects to the internet through a different UPF.

### Step 1
- Subscribe the User Equipments (UEs) to the graphical user interface of free5GC. To access the interface, simply open your web browser and navigate to port 5000, specifying either 'localhost' or the desired IP address.

- Login: admin

  Pass: free5gc

- In the panel that opens in a new subscription, you should add a UE with the IMSI and the slice data (SD and SSD) below:

<p align="center">
  <a href="https://i.ibb.co/9p3FSJd/ue01-sub.png">
    <img src="https://i.ibb.co/9p3FSJd/ue01-sub.png" width="1000" alt="free5gc-conf01">
    
  </a>
</p>

<p align="center">
  <a href="https://i.ibb.co/5c4ZqfJ/ue02-sub.png">
    <img src="https://i.ibb.co/5c4ZqfJ/ue02-sub.png" width="1000" alt="free5gc-conf02">
    
  </a>
</p>

### Step 2
After run the Docker Compose, execute the following command to start the two UEs.:
```bash
# Start the first UE
docker exec ueransim-ue bash -c "./nr-ue -c ./config/uecfg.yaml &"

#Start the second UE
docker exec ueransim-ue bash -c "./nr-ue -c ./config/uecfg2.yaml &"
```

### Step 3
Execute the tcpdump command on the UPFs to monitor the traffic passing through each of them.

```bash
# Run tcpdump on UPF
docker exec upf bash -c "tcpdump"

# Run tcpdump on UPF-2
docker exec upf-2 bash -c "tcpdump"
```
### Step 4
Generate traffic from the UEs by executing a ping to the Google server.

```bash
# Ping on the interface of the first UE (uesimtun0)
docker exec ueransim-ue bash -c "ping google.com -I uesimtun0"

# Ping on the interface of the second UE (uesimtun1)
docker exec ueransim-ue bash -c "ping google.com -I uesimtun1"
```
### Conclusion

When we run the ping from UE 1's interface, we observe this traffic passing through UPF-1. However, when we run the ping from UE 2's interface, we see the traffic passing through UPF-2.









## Reference
- https://github.com/open5gs/nextepc/tree/master/docker
- https://github.com/abousselmi/docker-free5gc
