# Installation & Administration Guide

## Index

* [Installation](#installation)
* [Usage](#usage)
* [Packaging](#packaging)
* [Configuration] (#configuration)

## <a name="installation"/> Installation
There are three ways of installing the Ultralight 2.0 Agent: cloning the Github repository, using the RPM or using Docker.
Regardless of the installation method, there are some middlewares that must be present, as a prerequisite for the component
installation (no installation instructions are provided for these middlewares):

* A MQTT v3.1 Broker is needed for the MQTT Binding to work. Both [Mosquitto](http://mosquitto.org/) and [Rabbit MQ](https://www.rabbitmq.com/)
(with the MQTT plugin activated) have been tested for this purpose.

* A [MongoDB](https://www.mongodb.com/) instance (v3.2+) is required for those IoT Agents configured to have persistent storage.
An in-memory storage repository is also provided for testing purposes.

* The IoT Agent purpose is to connect devices (using a certain southbound device protocol) and NGSI endpoints (typically
 a NGSI Context Broker, like [Orion](https://github.com/telefonicaid/fiware-orion)), so an accessible Context Broker is also required.
IoT Agents were tested with v0.26.0 (higher versions should also work).

Please, follow the links to the official Web Pages to find how can you install each of the middlewares in your environment.

The following sections describe each installation method in detail.

#### Cloning the Github repository

Clone the repository with the following command:
```
git clone https://github.com/telefonicaid/iotagent-ul.git
```

Once the repository is cloned, from the root folder of the project execute:
```
npm install
```
This will download the dependencies for the project, and let it ready to the execution.

When the component is executed from a cloned Github repository, it takes the default config file that can be found
in the root of the repository.

#### Using the RPM
To see how to generate the RPM, follow the instructions in [Packaging](#rpm).

To install the RPM, use the YUM tool:
```
yum localinstall --nogpg <rpm-file_name>
```

Be aware that the RPM installs linux services that can be used to start the application, instead of directly calling
the executable (as explained in the section [Usage](#usage).

When this option is used, all the files are installed under the `/opt/iotaul` folder. There you can find the `config.js`
file to configure the service. Remember to restart the service each time the config file has changed.

#### Using Docker
There are automatic builds of the development version of the IOTAgent published in Docker hub. In order to install
using the docker version, just execute the following:
```
docker run -d --link orion:orion --link mosquitto:mosquitto --link mongo:mongo -p 7896:7896 -p 4041:4041 telefonicaiot/iotagent-ul
```
As you can see, the Ultralight 2.0 (as any other IOTA) requires some docker dependencies to work:

* **mongo**: Mongo database instance (to store provisioning data).
* **orion**: Orion Context Broker.
* **mosquitto**: Mosquitto MQTT broker, to deal with MQTT based requests.

In order to link them, deploy them using docker and use the option `--link` as shown in the example. You may also want to
map the external IOTA ports, for external calls: 4041 (Northbound API) and 7896 (HTTP binding).

#### Build your own Docker image
There is also the possibility to build your own local Docker image of the IOTAUL component.

To do it, follow the next steps once you have installed Docker in your machine:

1. Navigate to the path where the component repository was cloned.
2. Launch a Docker build
    * Using the default NodeJS version of the operating system used defined in FROM keyword of Dockerfile:
    ```bash
    sudo docker build -f Dockerfile .
    ```
    * Using an alternative NodeJS version:
    ```bash
    sudo docker build --build-arg NODEJS_VERSION=0.10.46 -f Dockerfile .
    ```

## <a name="usage"/> Usage

#### Github installation
In order to execute the IOTAgent, just issue the following command from the root folder of the cloned project:
```
bin/iotagent-ul [config file]
```
The optional name of a config file is optional and described in the following section.

#### RPM installation
The RPM installs a linux service that can be managed with the typical instructions:
```
service iotaul start

service iotaul status

service iotaul stop
```

In this mode, the log file is written in `/var/log/iotaul/iotaul.log`.

#### Docker installation
The Docker automatically starts listening in the API ports, so there is no need to execute any process in order to
have the application running. The Docker image will automatically start.

## <a name="packaging"/> Packaging
The only package type allowed is RPM. In order to execute the packaging scripts, the RPM Build Tools must be available
in the system.

From the root folder of the project, create the RPM with the following commands:
```
cd rpm
./create-rpm.sh -v <version-number> -r  <release-number>
```
Where `<version-number>` is the version (x.y.z) you want the package to have and `<release-number>` is an increasing
number dependent un previous installations.

## <a name="configuration"/> Configuration

All the configuration for the IoT Agent resides in the `config.js` file, in the root of the application. This file
is a JavaSript file, that contains the following sections:

* **config.iota**: general IoT Agent configuration. This group of attributes is common to all types of IoT Agents, and
is described in the global [IoT Agent Library Documentation](https://github.com/telefonicaid/iotagent-node-lib#configuration).
* **config.mqtt**: configuration for the MQTT transport protocol binding of the IoTA (described in the following subsections).
* **config.http**: configuration for the HTTP transport protocol binding of the IoTA (described in the following subsections).
* **config.defaultKey**: default API Key, for devices lacking a provided Configuration.
* **config.defaultTransport**: code of the MQTT transport that will be used to resolve incoming commands and lazy attributes
 in case a transport protocol could not be inferred for the device.

#### MQTT Binding configuration
The `config.mqtt` section of the config file contains all the information needed to connect to the MQTT Broker from the
IoTAgent. The following attributes are accepted:

* **host**: Host where the MQTT Broker is located.
* **port**: Port where the MQTT Broker is listening
* **username**: User name for the IoTAgent in the MQTT broker, if authentication is activated.
* **password**: Password for the IoTAgent in the MQTT broker, if authentication is activated.

#### HTTP Binding configuration
The `config.http` section of the config file contains all the information needed to start the HTTP server for the HTTP
transport protocol binding. The following options are accepted:

* **port**: port where the southbound HTTP listener will be listening for information from the devices.

#### Configuration with environment variables
Some of the more common variables can be configured using environment variables. The ones overriding general parameters
in the `config.iota` set are described in the [IoTA Library Configuration manual](https://github.com/telefonicaid/iotagent-node-lib#configuration).

The ones relating specific Ultralight 2.0 bindings are described in the following table.

| Environment variable      | Configuration attribute             |
|:------------------------- |:----------------------------------- |
| IOTA_MQTT_HOST            | mqtt.host                           |
| IOTA_MQTT_PORT            | mqtt.port                           |
| IOTA_MQTT_USERNAME        | mqtt.username                       |
| IOTA_MQTT_PASSWORD        | mqtt.password                       |
| IOTA_HTTP_HOST            | http.host                           |
| IOTA_HTTP_PORT            | http.port                           |
