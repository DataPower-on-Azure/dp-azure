# Migration guide for DataPower from local Docker deployment to conatainerized pods running on OpenShift

## Local Deployment

**Pre-Reqs**

1. Install [Docker](https://docs.docker.com/get-docker/) or [Podman](https://podman.io/getting-started/installation) locally on your own device.
  - If you choose to use Podman, please change the appropriate commands below from `docker` to `podman`.
2. Configure your [GIT_ORG](./Configuration.md)
**Instructions**

1. Create a folder for the project.
2. Create the following subdirectories inside this folder.
  - config
  - local
  - certs
3. Change the permissions on these sub directories.
  - Mac 
    ```
    chmod -R 777 config local certs
    ```
  - Windows 
    ```
    icacls config local certs /grant *S-1-1-0:F
    ```
    _Note: You may have to use the full path on Windows to correctly change the permissions on the directories. We haven't had the ability to test this yet._
4. Pull the DataPower Docker image .
   ```
   docker pull icr.io/integration/datapower/datapower-limited:10.0.4.0
   ```
  - You may have to Create a [DockerID](https://hub.docker.com/).
  - And then [Login to Docker](https://docs.docker.com/engine/reference/commandline/login/).
5. Create and run the the DataPower container with an interactive shell from the local Docker image with volume mounts.
  - Make sure you are in the directory that you created that contains the "config", "local", and "certs" folders.
  - Mac
    ```
    docker run -it --name datapower \
    -v $(pwd)/config:/opt/ibm/datapower/drouter/config \
    -v $(pwd)/local:/opt/ibm/datapower/drouter/local \
    -v $(pwd)/certs:/opt/ibm/datapower/root/secure/usrcerts \
    -e DATAPOWER_ACCEPT_LICENSE="true" \
    -e DATAPOWER_INTERACTIVE="true" \
    -p 9090:9090 \
    -p 8001:8001 \
    icr.io/integration/datapower/datapower-limited:10.0.4.0
    ```
    _Note: Exposing port 8001 is specific to the validation-flow.zip.  Expose the correct port if importing a different flow._
  - Windows (PowerShell)
    ```
    docker run -it --name datapower \
    -v ${PWD}/config:/opt/ibm/datapower/drouter/config \
    -v ${PWD}/local:/opt/ibm/datapower/drouter/local \
    -v ${PWD}/certs:/opt/ibm/datapower/root/secure/usrcerts \
    -e DATAPOWER_ACCEPT_LICENSE="true" \
    -e DATAPOWER_INTERACTIVE="true" \
    -p 9090:9090 \
    -p 8001:8001 \
    icr.io/integration/datapower/datapower-limited:10.0.4.0
    ```
    _Note: Exposing port 8001 is specific to the validation-flow.zip.  Expose the correct port if importing a different flow._
6. Enable the UI.
  - login: `admin`
  - Password: `admin`
  - ```
    configure terminal
    ```
  - ```
    web-mgmt
    ```
  - ```
    admin-state "enabled"
    ```
  - ```
    exit
    ```
7. Access the DataPower Gateway on [https://localhost:9090](https://localhost:9090) to import the validation-flow.zip file for demo purposes.
8. Once your configuration is complete, export the config as a zip file and save everything to your mounted volumes.
  - In the GUI, click 'Save Configuration'.
  - In the CLI enter ```write memory```
  - Optional:
    - In the GUI, export the zip file if any changes were made to a configuration.
9. Exit the datapower CLI
  - Type `exit` twice and then "control+C".
11. Ensure that the config, local, and certs subdirectories are no longer empty.
12. Stop and delete the Docker container as well as remove the pulled DataPower Docker image if you wish.
  - ```
    docker stop datapower
    ```
  - ```
    docker rm datapower
    ```
  - ```
    docker rmi icr.io/integration/datapower/datapower-limited:10.0.4.0
    ```

## OpenShift Deployment

**Pre-Reqs**

1. An OpenShift installation on a cloud provider.
2. Login credentials and url address to access the OpenShift UI.

**Instructions**

1. Clone and run the [datapower-operator-scripts](https://github.com/DataPower-on-Azure/datapower-operator-scripts) repo and follow its instructions to extract the necessary yaml files and deploy using your chosen method.
