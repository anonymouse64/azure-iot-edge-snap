# Azure IoT Edge as a snap

This repo contains code to build Azure IoT Edge as a snap to run on Ubuntu Core or any system that supports snaps.

**Note**: This snap is currently a work-in-progress and should be considered a prototype, and as such may break at any point in time.

The snap includes a version of docker, using slightly modified sources from the docker snap. Notably, it doesn't use the moby engine that is included with the normal distribution of Azure IoT Edge. This is so that the docker engine works better with snapd confinement

# Building the snap

The snap is built with snapcraft. To build the snap:

```bash
git clone 
snapcraft
```

# Installing the snap

To install the snap use `--dangerous` for now:

```bash
sudo snap install --dangerous azure-iot-edge-ijohnson*.snap
```

After installation there will be 2 services in the snap, `dockerd` and `iotedged`. The `dockerd` daemon should can start fine, but currently won't have auto-connections for the various interfaces it needs to run, and as such is disabled by the install hook. So after installation, the interfaces need to be connected, then the daemon can be started. Additionally, the `iotedged` daemon should be disabled by the install hook because it doesn't have proper credentials inside the config file for `iotedged`. However, there is currently a bug with snapd where services that declare they have a unix socket they listen on are always started after the install hook. See this [forum post](https://forum.snapcraft.io/t/how-to-manage-services-with-sockets-timers/7904) for more info. The install still seems to succeed even though the `iotedged` service fails immediately. 

After the snap has been installed connect the following interfaces:

```bash
sudo snap connect azure-iot-edge-ijohnson:docker-support
sudo snap connect azure-iot-edge-ijohnson:firewall-control
sudo snap connect azure-iot-edge-ijohnson:home
```

Now start the `dockerd` service with:

```bash
sudo snap start --enable azure-iot-edge-ijohnson.dockerd
```

Now that `dockerd` is running successfully, you will need to provide your connection string obtained from Azure when you setup the hub so that `iotedged` can make a connection to Azure. The connection string (unquoted) should be put into a file and provide the file to the snap with `snap set` as shown below:

```bash
sudo snap set azure-iot-edge-ijohnson cs-file=/path/to/the/file
```

Note that the `home` interface is declared with `read: all` attribute, which means that when connected, root users in the snap can read from any user's home folder. This means that after connecting the home interface above, you can specify the `cs-file` as being anywhere in your `$HOME` directory.

This will add your connection string to the config file and automatically start the service running. After that you should be able to see the modules you added to the device running successfully with the `iotedge` command in the snap:

```bash
$ sudo azure-iot-edge-ijohnson.iotedge list
NAME                        STATUS           DESCRIPTION      CONFIG
edgeHub                     running          Up 11 seconds    mcr.microsoft.com/azureiotedge-hub:1.0
edgeAgent                   running          Up 34 seconds    mcr.microsoft.com/azureiotedge-agent:1.0
SimulatedTemperatureSensor  running          Up 26 seconds    mcr.microsoft.com/azureiotedge-simulated-temperature-sensor:1.0
```

# Known Issues
There is currently a bug where sometimes iotedged hangs forever at "Initializing modules" in the log. If you run into this issue, just restart iotedged with `snap restart azure-iot-edge-ijohnson.iotedged` and it should begin working again.

Additionally, while the snap is meant to use namespaced sockets, it seems to still somehow leak state with the host system outside of snap confinement, and as such it is recommended to remove all installations of docker, moby and iotedge from the host system before using the snap. A reboot may be necessary after removal to flush all systemd sockets, etc. out of the system.