apollo-mongodb
==============

Docker mongodb server generic image source. This is based on `ubuntu:14.04` image.

Environment
-----------

Before run see [apollo-coreos](https://github.com/wiliamsouza/apollo-coreos#environment) for instructions.

Volumes
-------

```
mkdir -p ~/.containers/apollo/mongodb
cp -R volumes/ ~/.containers/apollo/mongodb/
```

Image
-----

Build this image:

```
docker build -t $DOCKER_REGISTRY/apollo/mongodb:$APOLLO_TAG .
```

Pushing images
--------------

Push a image manually, this will preload the image to the cluster node:

```
IMAGE="${DOCKER_REGISTRY}/apollo/mongodb"
COREOS_IP=172.16.16.101
docker save $IMAGE | docker -H tcp://$COREOS_IP:2375 load
```

Push the image to local registry:

```
docker push $IMAGE:$APOLLO_TAG
```

Before push an image you need start a local registry `apollo-registry/README.md`
for instruction how to start a registry.


Starting service
----------------

Start `mongodb` service on the cluster:

```
cd systemd
ln -s mongodb.service mongodb@1.service
ln -s mongodb.service mongodb@2.service
ln -s mongodb.service mongodb@3.service
fleetctl start mongodb@1.service
fleetctl start mongodb@2.service
fleetctl start mongodb@3.service
```

Info about how to configure fleet `apollo-coreos/README.md#fleet`.

Container
---------

The commands here should be executed inside a cluster node.

```
eval `cat /etc/environment`
eval `cat /etc/env.d/apollo`
```

This image uses volumes and environment variables to control the mongodb server
configuration.

Volumes:

* `/etc/mongodb`: Change server configurations using it.
* `/var/lib/mongodb`: Data goes here.
* `/var/log/mongodb`: Access log from the container using it.

You pass with `-v` docker option. Don't forget to use absolute path here.
Shell access:

```
docker run --rm -p 80:80 -i \
-v /srv/containers/mongodb/volumes/log:/var/log/mongodb \
-v /srv/containers/mongodb/volumes/lib:/var/lib/mongodb \
-v /srv/containers/mongodb/volumes/etc:/etc/mongodb \
-t $DOCKER_REGISTRY/apollo/mongodb:$APOLLO_ENVIRONMENT /bin/bash
```

The command above will start a container give you a shell. Don't
forget to start the service running the `startup &` script.

Manual start:

```
docker run --name nginx -p 80:80 \
-v /srv/containers/mongodb/volumes/log:/var/log/mongodb \
-v /srv/containers/mongodb/volumes/lib:/var/lib/mongodb \
-v /srv/containers/mongodb/volumes/etc:/etc/mongodb \
-d $DOCKER_REGISTRY/apollo/mongodb:$APOLLO_ENVIRONMENT
```

The command above will start a container and return its ID.
