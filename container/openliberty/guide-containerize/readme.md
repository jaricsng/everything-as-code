
# OpenLiberty Microservices

You can easily deploy your microservices in different environments in a lightweight and portable manner by using containers. From development to production and across your DevOps environments, you can deploy your microservices consistently and efficiently with containers. You can run a container from a container image. Each container image is a package of what you need to run your microservice or application, from the code to its dependencies and configuration.

You’ll learn how to build container images and run containers using Docker for your microservices. You’ll construct Dockerfile files, create Docker images by using the docker build command, and run the image as Docker containers by using docker run command.

The two microservices that you’ll be working with are called system and inventory. The system microservice returns the JVM system properties of the running container. The inventory microservice adds the properties from the system microservice to the inventory. This guide demonstrates how both microservices can run and communicate with each other in different Docker containers.

## Packaging your microservices
To try out the microservices by using Maven, run the following Maven goal to build the system microservice and run it inside Open Liberty:

```bash
mvn -pl system liberty:run
```

Open another command-line session and run the following Maven goal to build the inventory microservice and run it inside Open Liberty:
```bash
mvn -pl inventory liberty:run
```

To access the inventory service, which displays the current contents of the inventory, see http://localhost:9081/inventory/systems.

The system service shows the system properties of the running JVM and can be found at http://localhost:9080/system/properties.

The system properties of your localhost can be added to the inventory at http://localhost:9081/inventory/systems/localhost.

After you are finished checking out the microservices, stop the Open Liberty servers by pressing CTRL+C in the command-line sessions where you ran the servers. Alternatively, you can run the liberty:stop goal in another command-line session:

```bash
mvn -pl system liberty:stop
mvn -pl inventory liberty:stop
```

Run the Maven package goal to build the application .war files from the start directory so that the .war files reside in the system/target and inventory/target directories.

```bash
mvn package
```

To learn more about RESTful web services and how to build them, see [Creating a RESTful web service](https://openliberty.io/guides/rest-intro.html) for details about how to build the system service. The inventory service is built in a similar way.

## Building your Docker images

Run the following command to download or update to the latest openliberty/open-liberty:kernel-java8-openj9-ubi Docker image:

```bash
docker pull openliberty/open-liberty:kernel-java8-openj9-ubi
```

Run the following commands to build container images for your application:

```bash
docker build -t system:1.0-SNAPSHOT system/.
docker build -t inventory:1.0-SNAPSHOT inventory/.
```

The -t flag in the docker build command allows the Docker image to be labeled (tagged) in the name[:tag] format. The tag for an image describes the specific image version. If the optional [:tag] tag is not specified, the latest tag is created by default.

To verify that the images are built, run the docker images command to list all local Docker images:

```bash
docker images
```

Or, run the docker images command with --filter option to list your images:
```bash
docker images -f "label=org.opencontainers.image.authors=Your Name"
```

Your two images, inventory and system, should appear in the list of all Docker images:

```bash
REPOSITORY          TAG                 IMAGE ID            CREATED              SIZE
inventory           1.0-SNAPSHOT        486788ecb85c        59 seconds ago       740MB
system              1.0-SNAPSHOT        b0d656b10bb0        About a minute ago   737MB
```

To access the application, point your browser to the http://localhost:9081/inventory/systems URL. An empty list is expected because no system properties are stored in the inventory yet.

Running your microservices in Docker containers

Now that you have your two images built, you will run your microservices in Docker containers:
```bash
docker run -d --name system -p 9080:9080 system:1.0-SNAPSHOT
docker run -d --name inventory -p 9081:9081 inventory:1.0-SNAPSHOT
```

Next, retrieve the system container’s IP address by using the system container’s name that is defined when it ran the Docker containers. Run the following command to retrieve the system IP address:
```bash
docker inspect -f "{{.NetworkSettings.IPAddress }}" system
```

You find the system container’s IP address:
```bash
172.17.0.2
```

Point your browser to http://localhost:9081/inventory/systems/[system-ip-address] by replacing [system-ip-address] with the IP address you obtained earlier. You see a result in JSON format with the system properties of your local JVM. When you visit this URL, these system properties are automatically stored in the inventory. Go back to http://localhost:9081/inventory/systems and you see a new entry for [system-ip-address].

> Populate system properties
```bash
curl http://localhost:9081/inventory/systems/172.17.0.2
```


## Testing the microservices
You can test your microservices manually by hitting the endpoints or with automated tests that check your running Docker containers.

```bash
Create the SystemEndpointIT class.
system/src/test/java/it/io/openliberty/guides/system/SystemEndpointIT.java
```

SystemEndpointIT.java
```bash
link:finish/system/src/test/java/it/io/openliberty/guides/system/SystemEndpointIT.java[]
```



The testGetProperties() method checks for a 200 response code from the system service endpoint.

```bash
Create the InventoryEndpointIT class.
inventory/src/test/java/it/io/openliberty/guides/inventory/InventoryEndpointIT.java
```

InventoryEndpointIT.java
```bash
link:finish/inventory/src/test/java/it/io/openliberty/guides/inventory/InventoryEndpointIT.java[]
```


- The testEmptyInventory() method checks that the inventory service has a total of 0 systems before anything is added to it.
- The testHostRegistration() method checks that the system service was added to inventory properly.
- The testSystemPropertiesMatch() checks that the system properties match what was added into the inventory service.
- The testUnknownHost() method checks that an error is raised if an unknown host name is being added into the inventory - service.
- The systemServiceIp variable has the same value as what you retrieved in the previous section when manually adding the - system service into the inventory service. This value of the IP address is passed in when you run the tests.


### Running the tests

```bash
mvn package
mvn failsafe:integration-test -Dsystem.ip=[system-ip-address] -Dinventory.http.port=9081 -Dsystem.http.port=9080
```

When you are finished with the services, run the following commands to stop and remove your containers:
```bash
docker stop inventory system
docker rm inventory system
```
