## Starting a simple Sync Gateway deployment

Sync Gateway's most basic configuration mechanism is the [bootstrap config](https://docs.couchbase.com/sync-gateway/current/configuration-schema-bootstrap.html).

When you spin up a Virtual Machine or a virtualized operating system environment, you can install Sync Gateway just like any other conventional application software (i.e. package managers such as apt, rpm, ~~yay~~, or just build the software).

Typically, installing daemonized software (programs meant to be server-processes) will result in the creation of a `systemd` unit for the software. [SystemD](https://systemd.io/) is the most widely used init system (PID 1) on Linux. Think of it like the OS's user-process-monitoring process. Whenever you install any such software via apt or dnf, the installer will do the following:

1. Register the process to be monitored by systemd.
2. Configure the process to automatically start when the machine starts - `systemctl enable <process-name>`
3. Start running the process at the current moment - `systemctl start <process-name>`

`systemd` will, from then onwards, start and monitor the process. The same semantics apply to Sync Gateway when you run it on a Virtual Machine. Sync Gateway will then also have its own home directory where you will find the configuration JSON file to edit.

However, in this task, we will work with containers as doing the above will be too time-consuming. But a few points to note:

* Sync Gateway will run successfully only if it has been configured beforehand.

    * On startup, Sync Gateway will typically try looking for its configuration file. If the file doesn't exist in the path it expected, it will exit. 
    
    * If the configuration file exists in hte expected path, but it isn't JSON or doesn't match the bootstrap spec, it will exit.

    * If the configuration file is valid JSON and matches the bootstrap spec, it will try contacting Couchbase Server at the specified connection string. It will attempt to connect, but if it fails repeatedly, Sync Gateway will exit again.

* Configuring Sync Gateway on Docker **differs** from doing it on Virtual Machines.

* Containers do not use `systemd` as their init system. For the container, the Sync Gateway process itself is the container. If this process dies, the container will be marked by Docker as 'crashed/killed' by the Docker daemon.

* On Virtual Machines, programs will log all errors and other information into `syslog` (used to be `/var/log/messages` earlier). From `systemd` onwards, `journalctl` became the standard command used to access these logs. 

* But for containers, any logging from the containerized-program can be sent to the log stores maintained by Docker. These can be accessed using the `docker logs` command.

---

### Task

* **If you have any running docker containers, please stop them to prevent port-mapping conflicts**

1. Run a Couchbase Server container (name it whatever you want, but I will use `mymob`). Ensure that you have access to the UI and other ports. Initialize the server with the following configuration:

```
Data  - 1024 MB
Index -  512 MB
Query -    0 MB
```

2. Create a bucket on Couchbase Server named `mobilebucket`.

3. Figure out the IP address of this container in the docker subnet. Note the IP address down somewhere.

---

### Initializing Sync Gateway

4. Use the following to pull the docker image:

```bash
docker pull couchbase/sync-gateway:4.0.2-enterprise
```

5. Run the container:

```bash
docker run -d -p 4984-4985:4984-4985 --name mobile-sgw couchbase/sync-gateway:4.0.2-enterprise
```

6. Observe what happens. Try getting the container's logs; figure out how to find docker container logs (either via terminal or from Docker Desktop).

7. From Docker logs, figure out the location of the config file that the the process inside the container was attempting to read.

---

### That didn't end well

The reason behind this failure is because in Docker containers, you have a chicken-and-egg situation. The container starts, but Sync Gateway's default configuration expects Couchbase Server to be running on localhost. The container's health is tied to the Sync Gateway process, and the Sync Gateway process will exit when it is unable to connect to a Couchbase Server instance. As a result, Sync Gateway process exits.

Therefore, we need to edit the configuration even before the container starts running. This is possible if you mount the config file from your system on the container as a shared file/volume.

Let us start writing a configuration file for Sync Gateway. 

**Challenge**: Try to get the default configuration on the Sync Gateway docker instance. This is not mandatory to perform.

8. Create the required configuration file for sync gateway. Save the following json content as a file named `config.json`:

```json
{
  "bootstrap": {
    "server": "couchbase://<cb_server_ip>",
    "username": "Administrator",
    "password": "password",
    "server_tls_skip_verify": true,
    "use_tls_server": false
  },
  "logging": {
    "console": {
      "enabled": true,
      "log_level": "info",
      "log_keys": ["*"]
    }
  }
}
```

9. Delete the Sync Gateway container. We will recreate it now, whilst sharing the volume.

10. Finally, share the file and run the container:

```bash
docker run -d -p 4984-4985:4984-4985 --name mobile-sgw -v ./conf:<config-file-path.json couchbase/sync-gateway:4.0.2-enterprise 
```

11. Send a REST API request to Sync Gateway:

```bash
curl 127.0.0.1:4984
```

But using curl will get tedious. We will install Postman for the next tasks.
