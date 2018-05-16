# sample

HelloWorld java program as sample
*************************************  Task- 1  (Continuous Integration)  CI *******************************************************
Job-1 Continuous Integration(CI)
****************************************************************************************************************************
using git,jenkins,nexus
for creating .jar file( use maven deploy command to create .jar file)
upload that jar file to nexus Repository.

Output should be: maven-snapshot content ur .jar file and .pom file

On ubuntu


install git
create sample project in git(also create sample repo in github.com)
install jenkins from digitalocean
install nexus(http://www.sonatype.org/nexus/2017/01/25/how-to-install-latest-sonatype-nexus-3-on-linux/)
get the hello-world sample java program from git(mkdir sample,cd sampleapp,git clone https://github.com/DeepakOhol/sample.git)
mkdir sample,cd sample  mv sampleapp (space)src/ (space)target/ (space)pom.xml (space) /home/user/sample
cd sample/         .........it will shows you that "something like nothing to .....initial .....something like that"
git status
git add .
git commit -m "put any message like initial project push"
git push origin master
sudo vi pom.xml    .......make following changes after"<dependencies>
    <dependency>
      <groupId>junit</groupId>
      <artifactId>junit</artifactId>
      <version>3.8.1</version>
      <scope>test</scope>
    </dependency>
  </dependencies>" part and add following to it
#################
 <distributionManagement>
          <repository>
                  <id>devrepo</id>
                  <name>Internal Repository</name>
                  <url>http://192.168.56.102:8081/#browse/browse/components:maven-snapshots</url>           ......................Put here, nexus Component URL
          </repository>

           <snapshotRepository>
                        <id>Snapshots</id>
                             <name>snapshotRepo</name>
                                  <url>http://192.168.56.102:8081/repository/maven-snapshots/</url>                                  ......................Put here, nexus Repository URL
                                   </snapshotRepository>
                           </distributionManagement>
#################
save and close
and push that file to git

and also add following lines to the file(sudo vi /usr/share/maven/conf/settings.xml)

inside<servers> tag add following lines:
<servers>

<server>
       <id>Snapshots</id>
       <username>admin</username>                                        .........this is ur nexus
       <password>admin123</password>                                   .........this is ur nexus default passwd
    </server>
</servers>

###############################



Now goto jenkins dashboard
open ur project(here my project name is "DevOps")
goto section "Source code mgmt"
select radio button git
                Repositories 
                                  Repository URL https://github.com/mohitepramod/sample.git
                                  Credentials username and passwd
               Branch to build:
                             Branch Specifier(blank for 'any') */master 
               Repository Browser(Auto)
###############################################

put following inside jenkins configure/build section
#############################################
mvn deploy                                                                   .........used for storing .jar file to nexus and we already made some changes to the file "sudo vi /usr/share/maven/conf/settings.xml"and "pom.xml" file
java -cp sample-0.0.1-SNAPSHOT.jar code.main.App    ..........currently no need to run this command manually on terminal
########## up to here ############

where ,
               sample-0.0.1-SNAPSHOT.jar  is the jar file which is created 
               code.main.App    is the main class name from POM.xml file

click on save .....It will save that.
click on "back to dashboard" which is available on top left corner
now click on "Build Now" button.
and see the console output.
finally it will gives u o/p as "Finished:SUCCESS"


Now goto nexus dashboard and goto component--->maven-snapshots--->click on that--->here u will see ur application folder with .jar and .pom files

Task completion is Done.
***************************************************** Task-1 Ends ********************************************************************************************

**********************************************************************************************************
Task-2   (Continuous Deployment) CD
**********************************************************************************************************


Now it comes to CD(Continuous Deployment Part):
scenerio:
                 output of Task-1(stored .jar file into nexus) should fetch that .jar file and put it into docker container
                 and execute that jar file in docker container()

first install docker in current Virtual machine(I am using 192.168.56.102 local machine) using following link:
https://www.digitalocean.com/community/tutorials/how-to-install-and-use-docker-on-ubuntu-16-04

On jenkins add new project named as"DevopsDeploy" and don't change any other section except "Build--->Execute shell" section.
Now, we need to get the .jar file freom nexus repo, for doing the same we need to go towards jenkins and do the following in build----->execute shell :
wget http://192.168.56.102:8081/repository/maven-snapshots/Sample/sample/0.0.1-SNAPSHOT/sample-0.0.1-20171102.060539-1.jar

after hit on save button


now, click on "Build Now" which is available on left pane, it will start executing job.we can see the executing job in consoleoutput.
there we can find out whether our job get successfully executed or not.
If u got response "SUCCESS" ,then ur job got executed successfully. and .jar file is coppied into following location:
"/var/lib/jenkins/workspace/ur_project_name(DevOpsDeploy)"   ......Here, "DevOpsDeploy" is my jenkins project name.

Now, Docker comes in to picture.
we need to (create)write one Dockerfile
sudo vim Dockerfile and paste the following content:
########## Dockerfile content ###################
FROM ubuntu:14.04
RUN ls
COPY . /
RUN apt-get update && \

    apt-get upgrade -y && \

    apt-get install -y  software-properties-common && \

    add-apt-repository ppa:webupd8team/java -y && \

    apt-get update && \

    echo oracle-java7-installer shared/accepted-oracle-license-v1-1 select true | /usr/bin/debconf-set-selections && \

    apt-get install -y oracle-java8-installer && \

    apt-get clean
RUN java -cp sample-0.0.1-20171102.060539-1.jar code.main.App

###############################################


explanation of content of Dockerfile:
first line is downloading the ubuntu 14.01 image
2nd line is for executing "ls" command
and for run the .jar file of java,there should be java installed on that docker container that's why we are installing java 8 (on docker container)
3rd line is for copying our .jar file to docker container's root directory
so from 4th line to 10th line (installation of java steps)
on 11th line we are executing our .jar file.

explanation of 11th line from Dockerfile is:--->
               RUN java -cp sample-0.0.1-20171102.060539-1.jar code.main.App

                     where,
                                    sample-0.0.1-20171102.060539-1.jar  ------> ur .jar file name
                                     code.main.App                 -------->     This name u can take from pom.xml file(value of tag <mainClass>code.main.App</mainClass>)



That's completed our Dockerfile.
Now, we need to execute following commands(to run Dockerfile):
goto project's workspace(here, my project workspace is "/var/lib/jenkins/workspace/ur_project_name(DevOpsDeploy)")

slave@salve-virtualbox:/var/lib/jenkins/workspace/DevOpsDeploy$ docker build .(dot) (hit enter)          .............This will execute our Dockerfile and gives docker-image id at the bottom(when it runs successfully.) 
                       
                   You will get output like:
                                                Sending build context to Docker daemon   5.12kB
                                                Step 1/5 : FROM ubuntu:14.04
                                                                    ---> dea1945146b9
                                                Step 2/5 : RUN ls
                                                                     ---> Using cache
                                                                     ---> 8fa205a97ec9
                                                Step 3/5 : COPY . /
                                                                     ---> afc0f7313586
                                                Step 4/5 : RUN apt-get update &&     apt-get upgrade -y &&     apt-get install -y  software-properties-common &&     add-apt-repository ppa:webupd8team/java -y &&     apt-get update &&     echo oracle-java7-installer shared/accepted-oracle-license-v1-1 select true | /usr/bin/debconf-set-selections &&     apt-get install -y oracle-java8-installer &&     apt-get clean
                                                                     ---> Running in 136306ca5218
                                                                   Get:1 http://security.ubuntu.com trusty-security InRelease [65.9 kB]
                                                                   Get:2 http://security.ubuntu.com trusty-security/universe Sources [77.9 kB]
                                                                   Ign http://archive.ubuntu.com trusty InRelease
                                                                   Get:3 http://security.ubuntu.com trusty-security/main amd64 Packages [852 kB]
                                                                   Get:4 http://archive.ubuntu.com trusty-updates InRelease [65.9 kB]
                                                                                                   :
                                                                                                   :
                                                                                                   :
                                                                                                   :
                                            Step 5/5 : RUN java -cp sample-0.0.1-20171102.060539-1.jar code.main.App
                                                                 ---> Running in 569b24323081
                                                                         Hello World!
                                                                 ---> 3d19b9a98821
                                                                Removing intermediate container 569b24323081
Successfully built 3d19b9a98821
######################################## here, we will get docker image id "Successfully built 3d19b9a98821", i.e  docker image id is "3d19b9a98821" ##################################

slave@salve-virtualbox:/var/lib/jenkins/workspace/DevOpsDeploy$ sudo docker run -itd docker_image_id(3d19b9a98821)      (hit enter)
         Output:
                      [sudo] password for slave: 
                      cc4002330d4390907bbcd47558a21892f120660caa23e31142655918c3573ddd                            ...................after running that command, we will get new docker_image_id for entering into docker-containers root directory.


slave@salve-virtualbox:/var/lib/jenkins/workspace/DevOpsDeploy$ sudo docker exec -it new_docker_image_id(cc4002330d4390907bbcd47558a21892f120660caa23e31142655918c3573ddd)  /bin/bash     (hit enter)

after completion of execution of the above command you will entered into Docker container's root directory.

Output of above command as follows:
        root@cc4002330d43:/#
        root@cc4002330d43:/# ls
        output:
                        Dockerfile       boot      etc      lib    media    opt     root       sample-0.0.1-20171102.060539-1.jar   ....................This is our .jar file
                        srv  tmp  var   bin     dev   home  lib64  mnt    proc  run   sbin    sys  usr

