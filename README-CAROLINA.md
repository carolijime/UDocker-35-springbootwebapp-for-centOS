#Create Centos image (need the tail -f command for it to not stop runnning
docker run -d centos tail -f /dev/null

#SSH Into a Running Docker Container (container name e.x naughty_brattain)
docker exec -it <container name> bash 

####in the SSH commands
# current user
whoami
# things that are running inside the image
ps -ef
# kill a process
kill -9 <process PID>

####Below instructions to install java so we can run a spring boot application (create an image with it inside)
#check if java is installed (inside the container's SSH shell)
java -version
#run below 2 commands so we are able to install the deprecated CetOS8 (https://forketyfork.medium.com/centos-8-no-urls-in-mirrorlist-error-3f87c3466faa)
sed -i -e "s|mirrorlist=|#mirrorlist=|g" /etc/yum.repos.d/CentOS-*
sed -i -e "s|#baseurl=http://mirror.centos.org|baseurl=http://vault.centos.org|g" /etc/yum.repos.d/CentOS-*
#Install java (inside the container's SSH shell) - press 'y' and the 'y' when being asked
yum install java 

#******Below how to run spring boot app from Docker*********
##1)Create a JAR from your Maven project (using IntelliJ)
#Open your Maven project in IntelliJ IDEA (File → Open).
#Make sure that the Maven tool window is visible by going to View → Tool Windows → Maven.
#Expand your project in the tree, expand Lifecycle and double-click on package.
#IntelliJ will run the Maven package phase, and you’ll see the output in another window below.
#Your JAR file will be available at target/your-app-1.0.jar!
#Job done. 😄

##2)Copy jar file from Windows to ubuntu (do this in ubuntu console)
# locate window file inside ubuntu (folder below after /c/ might change
 cd /mnt/c/Users/carol/DevPro/IdeaProjects/Docker-Java-Udemy/35-springbootwebapp-for-centOS/target/
# copy/paste desire file (note: from and to paths might change) using the cp <copy-from-file> <paste-to-folder>
 cp /mnt/c/Users/carol/DevPro/IdeaProjects/Docker-Java-Udemy/35-springbootwebapp-for-centOS/target/spring-boot-web-0.0.1-SNAPSHOT.jar ~/docker/dockercontainers/35-springbootwebapp-for-centOS
 
 
##3)Create Dockerfile
#Create empy Dockerfile
touch Dockerfile
#Edit file (use vi editor)
vi Dockerfile
#Add below code to Dockerfile (content might change in future). Note: the -y in the java iinstallation command is for the yes answers to the installation of java
FROM centos

RUN sed -i -e "s|mirrorlist=|#mirrorlist=|g" /etc/yum.repos.d/CentOS-*
RUN sed -i -e "s|#baseurl=http://mirror.centos.org|baseurl=http://vault.centos.org|g" /etc/yum.repos.d/CentOS-*
RUN yum install -y java-11-openjdk

VOLUME /tmp
ADD /spring-boot-web-0.0.1-SNAPSHOT.jar myapp.jar
RUN sh -c 'touch /myapp.jar'
ENTRYPOINT ["java","-Djava.security.egd=file:/dev/./urandom","-jar","/myapp.jar"]

##4) build docker image (inside the folder where previously created Dockerfile is). It will create an image. You will get the image id, but we will use the tag to run it in next step
#Note: -t is for creating a tag. Command below defaults to the already created Dockerfile in the current folder
docker build -t spring-boot-docker .

##5) run docker image
docker run -d -p 8080:8080 spring-boot-docker 

#*****END: Below how to run spring boot app from Docker*********