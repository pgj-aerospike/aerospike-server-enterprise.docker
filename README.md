## Aerospike Server Enterprise Edition Dockerfile

This repository contains the Dockerfile for building a Docker image for running [Aerospike](http://aerospike.com). 

## Dependencies

- [debian:strech-slim](https://hub.docker.com/_/debian)

## Installation

1. Install [Docker](https://www.docker.io/).

2. Download from public [Docker Registry](https://index.docker.io/):

		docker pull aerospike/aerospike-server-enterprise


### Usage

The following will run `asd` with all the exposed ports forward to the host machine.

	sudo docker run -tid --name aerospike -p 3000:3000 -p 3001:3001 -p 3002:3002 -p 3003:3003 -v <DIRECTORY>:/etc/aerospike/ -e "FEATURE_KEY_FILE=/etc/aerospike/features.conf" aerospike/aerospike-server-enterprise
	
**NOTE:** Feature key file is mandatory for running aerospike server enterprise edition.

# Advanced Usage 

## Custom Configuration

There are two ways to configure Aerospike.

**Environment Variables**

You can provide environment variables to the running container via the `-e` flag. To set my default Namespace name to "aerospike-demo":

    docker run -e "NAMESPACE=aerospike-demo" aerospike/aerospike-server-enterprise ...

List of Environment Variables:

  * FEATURE_KEY_FILE - Default: /etc/aerospike/features.conf
  * SERVICE_THREADS - Default: Number of vCPUs
  * TRANSACTION_QUEUES - Default: Number of vCPUs
  * TRANSACTION_THREADS_PER_QUEUE - Default: 4
  * LOGFILE - Default: /dev/null, do not log to file, log to stdout
  * SERVICE_ADDRESS - Default: any
  * SERVICE_PORT - Default: 3000
  * HB_ADDRESS - Default: any
  * HB_PORT - Default: 3002
  * FABRIC_ADDRESS - Default: any
  * FABRIC_PORT - Default: 3001
  * INFO_ADDRESS - Default: any
  * INFO_PORT - Default: 3003
  * NAMESPACE - Default: test
  * REPL_FACTOR - Default: 2
  * MEM_GB - Default: 1, the unit is always `G` (GB)
  * DEFAULT_TTL - Default: 30d
  * STORAGE_GB - Default: 4, the unit is always `G` (GB)

See the [configuration reference](https://www.aerospike.com/docs/reference/configuration/index.html) for what each controls.

This is not compatible with using custom configuration files.

**Custom Conf File**


By default, `asd` will use the configuration file in `/etc/aerospike/aerospike.conf`, which is generated by the entrypoint script. Environment variables will have no effect on your custom configuration file. To provide a custom configuration, you should first mount a directory containing the file using the `-v` option for `docker`:

	-v <DIRECTORY>:/opt/aerospike/etc

Where `<DIRECTORY>` is the path to a directory containing your custom configuration file. Next, you will want to tell `asd` to use a configuration file from `/opt/aerospike/etc`, by using the `--config-file` option for `aerospike/aerospike-server-enterprise`:
 
	--config-file /opt/aerospike/etc/aerospike.conf

This will use tell `asd` to use the file in `/opt/aerospike/etc/aerospike.conf`, which is mapped to `<DIRECTORY>/aerospike.conf`.

A full example:

	docker run -tid -v <DIRECTORY>:/opt/aerospike/etc -v <DIRECTORY>:/etc/aerospike/ -e "FEATURE_KEY_FILE=/etc/aerospike/features.conf" --name aerospike -p 3000:3000 -p 3001:3001 -p 3002:3002 -p 3003:3003 aerospike/aerospike-server-enterprise /usr/bin/asd --foreground --config-file /opt/aerospike/etc/aerospike.conf

## access-address Configuration

In order for Aerospike to properly broadcast its address to the cluster or applications, the **access-address** needs to be set in the configuration file. If it is not set, then the IP address within the container will be used, which is not accessible to other nodes.

To specify **access-address** in aerospike.conf:

	network {
		service {
			address any                  # Listening IP Address
			port 3000                    # Listening Port
			access-address 192.168.1.100 # IP Address to be used by applications
																	 # and other nodes in the cluster.
		}
		...


## Persistent Data Directory

With Docker, the files within the container are not persisted. To persist the data, you will want to mount a directory from the host to the guest's `/opt/aerospike/data` using the `-v` option:

	-v <DIRECTORY>:/opt/aerospike/data

Where `<DIRECTORY>` is the path to a directory containing your data files.

A full example:

	docker run -tid -v <DIRECTORY>:/opt/aerospike/data --name aerospike -p 3000:3000 -p 3001:3001 -p 3002:3002 -p 3003:3003 -v <DIRECTORY>:/etc/aerospike/ -e "FEATURE_KEY_FILE=/etc/aerospike/features.conf" aerospike/aerospike-server-enterprise

## Persistent Lua Cache

Upon restart, your lua cache will become emptied. To persist the cache, you will want to mount a directory from the host to the guest's `/opt/aerospike/usr/udf/lua` using the `-v` option:

	-v <DIRECTORY_LUA>:/opt/aerospike/usr/udf/lua
	
Where `<DIRECTORY_LUA>` is the path to a directory used as a persistent lua cache directory.

In case you did modify the default lua path within the `mod-lua`-block in your server configuration, match the path accordingly:

	- v <DIRECTORY_LUA>:<YOUR_DEFINED_LUA_PATH>
	
Where `<DIRECTORY_LUA>` is the path to a directory used as a persistent lua cache directory and `<YOUR_DEFINED_LUA_PATH>` is the lua path set in your server configuration's `mod-lua`-block.

A full example:

	docker run -tid -v <DIRECTORY>:/opt/aerospike/data -v <DIRECTORY_LUA>:/opt/aerospike/usr/udf/lua --name aerospike -p 3000:3000 -p 3001:3001 -p 3002:3002 -p 3003:3003 -v <DIRECTORY>:/etc/aerospike/ -e "FEATURE_KEY_FILE=/etc/aerospike/features.conf" aerospike/aerospike-server-enterprise


## Clustering

Aerospike recommends using multicast clustering whenever possible, however, we are currently working to figure out how to best support multicast via Docker. For the time being, it will be best to setup Mesh Clustering. We are open to pull-requests with proposals on how to implement multicast for our Dockerfile.

### Mesh Clustering

Mesh networking requires setting up links between each node in the cluster. This can be achieved in two ways:

1. Define a configuration for each node in the cluster, as defined in [Network Heartbeat Configuration](http://www.aerospike.com/docs/operations/configure/network/heartbeat/#mesh-unicast-heartbeat).

2. Use `asinfo` to send the `tip` command, to make the node aware of another node, as defined in [tip command in asinfo](http://www.aerospike.com/docs/tools/asinfo/#tip).



# Supported Docker versions

This image is officially supported on Docker version 1.4.1.

Support for older versions (down to 1.0) is provided on a best-effort basis.

# User Feedback

## Issues

If you have any problems with or questions about this image, please contact us on the [Aerospike Forums](discuss.aerospike.com) or through a [GitHub issue](https://github.com/aerospike/aerospike-server-enterprise.docker/issues).


## Contributing

You are invited to contribute new features, fixes, or updates, large or small; we are always thrilled to receive pull requests, and do our best to process them as fast as we can.