Running a Spring Boot App with Docker
=====================================

Step 1: install docker
----------------------

I followed these [instructions](https://docs.docker.com/installation/ubuntulinux/#ubuntu-precise-1204-lts-64-bit) for Ubuntu 12.04.

In addition to that I also added my user to the 'docker' group. 

     sudo usermod -a -G docker $USER

This allows the user to run docker without having to type 'sudo' each time.

For the rest of the steps I mostly followed as described [here](http://blog.adaofeliz.com/2014/11/21/first-look-spring-boot-and-docker/)
with some minor variations because I was using the 'New Spring Starter' wizard in STS which does some of the pom setup for us.

Step 2: create a boot app
-------------------------

In STS use the 'New Spring Starter' wizard to create a new web app (select the 'web' starter dependency in the wizard).

I called my app 'spring-boot-docker' and also changed the 'artifact-id' to 'spring-boot-docker'.

You can add some stuff to your app if you want (a hello world page). I was content with what boot generated... which is an webserver that only serves up 404 error pages.

Step 3: dockerify the app
-------------------------

Create a 'docker' folder at the root of your project. 

Create a file 'docker/Dockerfile' and put this content in it:

      FROM dockerfile/java:oracle-java7
      ADD spring-boot-docker-0.0.1-SNAPSHOT.jar /opt/spring-boot-docker/
      EXPOSE 8080
      WORKDIR /opt/spring-boot-docker/
      CMD ["java", "-jar", "spring-boot-maven-docker.jar"]

Add this to the pom:

    <plugin>
        <artifactId>maven-resources-plugin</artifactId>
        <executions>
            <execution>
                <id>copy-resources</id>
                <phase>validate</phase>
                <goals>
                    <goal>copy-resources</goal>
                </goals>
                <configuration>
                    <outputDirectory>${basedir}/target/</outputDirectory>
                    <resources>
                        <resource>
                            <directory>docker</directory>
                            <filtering>true</filtering>
                        </resource>
                    </resources>
                </configuration>
            </execution>
        </executions>
    </plugin>

Step 4: Create docker image
---------------------------

First we need to build our stuff with maven. From the root of the project:

     mvn clean install
     
Then we can build the image

     cd target
     docker build -t spring-boot-docker .
     
Step 5: Run the image
---------------------

Check that the image was create and determine its id:

     docker images
     
This shows you a list of images find the 'spring-boot-docker' one and write down its ID to use in the
next command:

     docker run -p 8080:8080 9969adf59c45
     
You can now connect to http://localhost:8080/

Note: unless you actually added something to your app (like a hello world page) you'll 
get a 404 error page.
