#Building Docker Images

Spring Boot applications can be containerized by packaging them into Docker images. 
A typical Spring Boot fat jar can be converted into a Docker image by adding just a few lines to a Dockerfile that can be used to build the image. 
However, there are various downsides to copying and running the fat jar as is in the docker image. 
There’s always a certain amount of overhead when running a fat jar without unpacking it, and in a containerized environment this can be noticeable. 
The other issue is that putting your application’s code and all its dependencies in one layer in the Docker image is sub-optimal. 
Since you probably recompile your code more often than you upgrade the version of Spring Boot you use, it’s often better to separate things a bit more. 
If you put jar files in the layer before your application classes, Docker often only needs to change the very bottom layer and can pick others up from its cache.


###Create layers.xml 
* Create layers.xml under source folder ( check layers.xml and pom.xml for config)



##Build using


##1. Layering Docker Images

To make it easier to create optimized Docker images that can be built with a dockerfile, Spring Boot supports adding a layer index file to the jar. The layers.idx file lists all the files in the jar along with the layer that the file should go in. The list of files in the index is ordered based on the order in which the layers should be added. Out-of-the-box, the following layers are supported:

* dependencies (for regular released dependencies)
* spring-boot-loader (for everything under org/springframework/boot/loader)
* snapshot-dependencies (for snapshot dependencies)
* application (for application classes and resources)


````mvn clean install````
````java -Djarmode=layertools -jar target/spring-build-packs-0.0.1-SNAPSHOT.jar````



###1.Using Docker file

````

FROM adoptopenjdk:14-jre-hotspot as builder
WORKDIR application
ARG JAR_FILE=target/*.jar
COPY ${JAR_FILE} application.jar
RUN java -Djarmode=layertools -jar application.jar extract

FROM adoptopenjdk:14-jre-hotspot
WORKDIR application
COPY --from=builder application/dependencies/ ./
COPY --from=builder application/spring-boot-loader/ ./
COPY --from=builder application/snapshot-dependencies/ ./
COPY --from=builder application/application/ ./
ENTRYPOINT ["java", "org.springframework.boot.loader.JarLauncher"]

````

##2.Using Build Packs ( Spring Boot Version 2.3.1 or later)** 

````mvn spring-boot:build-image ````

#####using dive command to check how it placed the contents 

```` dive <jar-image> ```` 

If you don't have dive command, please follow the instructions in the link to install dive ( https://www.ostechnix.com/how-to-analyze-and-explore-the-contents-of-docker-images/ )
 
For more info, please check the below spring documentation.

layers (https://docs.spring.io/spring-boot/docs/2.3.2.BUILD-SNAPSHOT/maven-plugin/reference/html/#repackage-layers)
