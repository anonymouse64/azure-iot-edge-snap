# Azure IoT Edge as a snap

This repo contains code to build Azure IoT Edge as a snap to run on Ubuntu Core or any system that supports snaps.

The snap includes a version of docker, using slightly modified sources from the docker snap. Notably, it doesn't use the moby engine that is included with the normal distribution of Azure IoT Edge. This is so that the docker engine works better with snapd confinement

# Building the snap

The snap is built with snapcraft. To build the snap:

```bash
git clone 
snapcraft
```

# Installing the snap

To install the snap use devmode for now:

```bash
sudo snap install --devmode azure-iot-edge-ijohnson*.snap
```

After installation there will be 2 services in the snap, 1 for `dockerd` and one for `iotedged`. The `dockerd` daemon should be running fine, but the `iotedged` daemon will fail because it doesn't have proper credentials inside the config file for `iotedged`. The `iotedged` service is supposed to be disabled from the install hook so that it doesn't attempt to run at all, but there is currently a bug with snapd where services that declare they have a unix socket they listen on are always started after the install hook. See this [forum post](https://forum.snapcraft.io/t/how-to-manage-services-with-sockets-timers/7904) for more info.

After the snap has been installed connect the following interfaces:

```bash
sudo snap connect azure-iot-edge-ijohnson:docker-cli azure-iot-edge-ijohnson:docker-daemon
sudo snap connect azure-iot-edge-ijohnson:support
```

(note that there are other interfaces that the snap plugs, but there's issues with plugging those, namely that the device cgroup gets turned on and that breaks creating containers for some currently unknown reason, so for now just plug these)

After the interfaces have been connected you will need to provide your connection string obtained from Azure when you setup the hub. The connection string (unquoted) should be put into a file and provide the file to the snap with `snap set` as shown below:

```bash
sudo snap set azure-iot-edge-ijohnson cs-file=/path/to/the/file
```

This will add your connection string to the config file and automatically start the service running. After that you should be able to see the modules you added to the device running successfully with the `iotedge` command in the snap:

```bash
$ sudo azure-iot-edge-ijohnson.iotedge list
NAME                        STATUS           DESCRIPTION      CONFIG
edgeHub                     running          Up 11 seconds    mcr.microsoft.com/azureiotedge-hub:1.0
edgeAgent                   running          Up 34 seconds    mcr.microsoft.com/azureiotedge-agent:1.0
SimulatedTemperatureSensor  running          Up 26 seconds    mcr.microsoft.com/azureiotedge-simulated-temperature-sensor:1.0
```
