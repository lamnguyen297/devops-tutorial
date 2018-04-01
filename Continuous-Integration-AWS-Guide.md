# CI/CD With Git Jenkins Artifactory And ELK Stack

## Connect to AWS

```
$ ssh -i /Users/apple/Workspace/90-Config/10-KeyPair/steve_nguyen_key_pair.pem ec2-user@ec2-54-149-77-190.us-west-2.compute.amazonaws.com
```

## Installation and Configuration

* Downloads 

```
$ mkdir downloads
```
* Tomcat
```
$ wget http://mirrors.viethosting.com/apache/tomcat/tomcat-8/v8.5.29/bin/apache-tomcat-8.5.29.tar.gz
$ tar -xvzf apache-tomcat-8.5.29.tar.gz
```
* Maven
```
$ wget http://mirrors.viethosting.com/apache/maven/maven-3/3.5.3/binaries/apache-maven-3.5.3-bin.zip
$ unzip apache-maven-3.5.3-bin.zip
```

* Elastisearch
```
$ wget https://artifacts.elastic.co/downloads/elasticsearch/elasticsearch-6.2.3.zip
$ unzip elasticsearch-6.2.3.zip
```

* Kibana
```
$ wget https://artifacts.elastic.co/downloads/kibana/kibana-6.2.3-linux-x86_64.tar.gz
$ tar -xvzf kibana-6.2.3-linux-x86_64.tar.gz
```

* Logstash
```
$ wget https://artifacts.elastic.co/downloads/logstash/logstash-6.2.3.zip
$ unzip logstash-6.2.3.zip
```

* Jfrog Artifactory
    * Download [Jfrog Artifactory](https://api.bintray.com/content/jfrog/artifactory/jfrog-artifactory-oss-$latest.zip;bt_package=jfrog-artifactory-oss-zip)
```
$ wget https://api.bintray.com/content/jfrog/artifactory/jfrog-artifactory-oss-$latest-sources.tar.gz;bt_package=jfrog-artifactory-oss-zip
$ unzip jfrog-artifactory-oss-5.9.1.zip
```

```
$ mkdir devops
$ cd devops
$ cp -R ../downloads/apache-maven-3.5.3 /usr/lib/
$ cp -R ../downloads/apache-tomcat-8.5.29 .
$ cp -R ../downloads/elasticsearch-6.2.3 .
$ cp -R ../downloads/kibana-6.2.3-linux-x86_64 .
$ cp -R ../downloads/logstash-6.2.3 .
$ cp -R ../downloads/jfrog-artifactory-oss-5.9.1/ .
```

### Config Tomcat

```
$ cd devops/apache-tomcat-8.5.29/conf
$ vi tomcat-users.xml
```

```xml
<role rolename="tomcat"/>
<role rolename="manager-gui"/>
<role rolename="manager-script"/>
<user username="tomcat" password="Aviva@Tomcat" roles="tomcat"/>
<user username="manager" password="Aviva@Manager" roles="tomcat,manager-gui"/>
<user username="jenkins" password="Aviva@Jenkins" roles="manager-script"/>
```

* Config to access from differrent host

```
$ vi  ../webapps/manager/META-INF/context.xml
```

```xml
<Context antiResourceLocking="false" privileged="true" >
  <Valve className="org.apache.catalina.valves.RemoteAddrValve"
         allow="^.*" />
</Context>
```
```
$ cd ..
$ cd bin
$ chmod +x *.sh
$ ./catalina.sh start
$ tail ../logs/catalina.out
```

### Config Maven

```
$ cd
$ vi .bash_profile
```

```
# java setting
export JAVA_HOME=/Library/Java/JavaVirtualMachines/jdk1.8.0_25.jdk/Contents/Home
export PATH=$PATH:$JAVA_HOME/bin

#maven setting
export M2_HOME=/home/ec2-user/devops/apache-maven-3.5.3
export M2=$M2_HOME/bin
export PATH=$PATH:$M2
```

```
$ more .bash_profile
$ source ~/.bash_profile
```


### Install Java 8

```
$ tar -xvzf jdk-8u161-linux-x64.tar.gz
$ sudo cp -R jdk1.8.0_161 /usr/lib/jvm/
```

* Set Java environment
```
$ cd
$ ls -all
$ vi .bash_profile
```
```
# java setting
export JAVA_HOME=/usr/lib/jvm/jdk1.8.0_161
export PATH=$PATH:$JAVA_HOME/bin
```
```
$ more .bash_profile
$ source ~/.bash_profile
```
* Enter the following commands to inform about the Java's location. 
```
$ sudo update-alternatives --install "/usr/bin/java" "java" "/usr/lib/jvm/jdk1.8.0_161/bin/java" 0
$ sudo update-alternatives --install "/usr/bin/javac" "javac" "/usr/lib/jvm/jdk1.8.0_161/bin/javac" 0
$ sudo update-alternatives --set java /usr/lib/jvm/jdk1.8.0_161/bin/java
$ sudo update-alternatives --set javac /usr/lib/jvm/jdk1.8.0_161/bin/javac
```

* To verify the setup enter the following commands 
```
$ update-alternatives --list java
$ update-alternatives --list javac
```
### Install and Configuration Jfrog Artifactory
```
$ cd devops/artifactory-oss-5.9.1/bin
$ ./artifactory.sh
```

### Install and Configuration Jenkins

```
$ sudo wget -O /etc/yum.repos.d/jenkins.repo https://pkg.jenkins.io/redhat-stable/jenkins.repo
$ sudo rpm --import https://pkg.jenkins.io/redhat-stable/jenkins.io.key
$ sudo yum install jenkins
$ sudo service jenkins start
```

* Jenkins Home: `/var/lib/jenkins/`

* Go http://ec2-54-149-77-190.us-west-2.compute.amazonaws.com:8080/

![](CI_CD_Guid_Images/Jenkins-CI-step0.png)

* Get Administrator password 
```
$ sudo more /var/lib/jenkins/secrets/initialAdminPassword
```

`2d6d611f7d5042c38162f563834d392b`

* Plugins for Jenkins

`Folders`, `OWASP Markup Formatter`, `build timeout`, `Credentials Binding`, `Timstamper`, `Workspace Cleanup`, `Pipeline`, `GitHub Branch Source`, `Pipline: GitHub Groovy Libraries`, `Pipline: Stage View`, `Git`, `SSH Slaves`, `Matrix Authorization Strategy`, `PAM Authentication`, `LDAP`, `Email Extension`, `Mailer`, 

* Setup admin user

```
Username:	lamnguyen  
Password:	Aviva@Jenkins
Confirm password:	Aviva@Jenkins
Full name:	Lam Nguyen
E-mail address:thanhlam297@gmail.com
```

```
$ sudo vi /etc/sysconfig/jenkins
```
* change JENKINS_PORT="8090"

```
$ sudo service jenkins status
$ sudo service jenkins stop
$ sudo service jenkins start
$ sudo service jenkins restart
```

* `Manage Jenkins` -> `Manage Plugins` -> `Available` tab to install plugins below:

    Conditional Buildstep, Deploy to container, Environment Injection, Git Parameter, SonarQube Scanner

* On left navigation, `Manage Jenkins` -> `Global Tool Configuration`:

* Config JDK
![](CI_CD_Guid_Images/Jenkins-CI-step01.png)
    - JDK Name : JAVA
    - Get JAVA_HOME and pate to JAVA_HOME field
    ```
    $ echo $JAVA_HOME
    /usr/lib/jvm/jdk1.8.0_161
    ```
* Config Maven
![](CI_CD_Guid_Images/Jenkins-CI-step02.png)
    - Maven Name : maven
    - Get maven hom and pate to MAVEN_HOME field
    ```
    $ echo $M2_HOME
    /home/ec2-user/devops/apache-maven-3.5.3
    ```
* Config Jenkins with GitHub

    * Generating a new SSH key
    ```
    $ ssh-keygen -t rsa -b 4096 -C "thanhlam297@gmail.com"
    $ eval "$(ssh-agent -s)"
    $ ssh-add -K ~/.ssh/id_rsa
    $ cd .ssh
    $ more id_rsa.pub
    ```

    * Install Git
    ```
    $ sudo yum install git
    ```
    ```
    $ sudo su jenkins
    $ cd
    $ ssh -T git@github.com
    $ git config --global user.name "jenkins"
    $ git config --global user.email thanhlam297@gmail.com
    ```
* Create New Job on Jenkins
![](CI_CD_Guid_Images/Jenkins-CI-step03.png)
![](CI_CD_Guid_Images/Jenkins-CI-step031.png)
![](CI_CD_Guid_Images/Jenkins-CI-step04.png)
![](CI_CD_Guid_Images/Jenkins-CI-step051.png)
![](CI_CD_Guid_Images/Jenkins-CI-step05.png)
![](CI_CD_Guid_Images/Jenkins-CI-step06.png)
![](CI_CD_Guid_Images/Jenkins-CI-step07.png)
![](CI_CD_Guid_Images/Jenkins-CI-step08.png)
![](CI_CD_Guid_Images/Jenkins-CI-step09.png)
![](CI_CD_Guid_Images/Jenkins-CI-step10.png)
![](CI_CD_Guid_Images/Jenkins-CI-step11.png)
![](CI_CD_Guid_Images/Jenkins-CI-step12.png)





