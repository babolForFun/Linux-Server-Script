
# INSTALL MULTIPLE TOMCAT INSTANCE IN LINUX SERVER BY SSH

## LEGEND

	PATH_KEY       = path of ssh_pem.pem file 
	PATH_TOMCAT    = folder with Tomcat instance
	KEY_SSH        = .pem file, certificate key to access into AWS
 	AWS_SERVER_IP  = ip of the machine you created on AWS - ex. ec2-xxx-xxx-xxx-xxx.eu-west-1.compute.amazonaws.com
	TOMCAT_[#name] = name of multiple instance - ex. TOMCAT_i1, TOMCAT_i2


#### AWS console
    Create new linux machine 
    Create a key and download the .pem file
    Save the SERVER_IP of the machines (or Elastic IP) 


#### TOMCAT
* Download [tomcat for linux](https://tomcat.apache.org/download-70.cgi "Tomcat") (Binary Distribution - tar.gz file)


#### CMD (as administrator)

* Install OpenSSH (Only if you are in a windows machine) follow this [OpenSSH tutorial](http://linuxbsdos.com/2015/07/30/how-to-install-openssh-on-windows-10/ "OpenSSH Tutorial")
	

* Change permission of the .pem file 
```sh
        $ chmod 600 [PATH_KEY]\KEY_SSH.pem 
```

* Copy *tomcat.tar.gz* folder from our machine to AWS machine by SSH (using port 22, remember colon at the end!)	
```sh
        $ scp -i "[PATH_KEY]\KEY_SSH.pem" "[PATH_TOMCAT]\apache-tomcat-x.x.xx.tar.gz" ec2-user@[AWS_SERVER_IP]: 
```
* Connect to AWS using ssh protocol
```sh
	    $ ssh -v -i [PATH_KEY]\KEY_SSH.pem ec2-user@AWS_SERVER_IP
```

* Once connected check if tomcat folder is in our home.  You should have something like *apache-tomcat-x.x.x.tar.gz*. If not repeat the SSH tranfer
```sh
	    $ ls -la
 ```

* Decompress apache tomcat tar file 
```sh
	    $ tar -zxvf apache-tomcat-x.x.x.tar.gz 
 ```

* Rename the file 
```sh	
	    $ mv apache-tomcat-x.x.x TOMCAT_i1
 ```

* Make a copy of tomcat folder (repeat this action as many times as how many tomcat instance you want to configure )
```sh	
	    $ cp -rf TOMCAT_i1 ./TOMCAT_i2
 ```

* Copy the folders into */opt/* (repeat for each instance)
```sh	
	    $ sudo mv ./TOMCAT_i1 /opt/
	    $ sudo mv ./TOMCAT_i2 /opt/
    	...
```

* Edit server configuration file (repeat for each instance)
```sh
	    $ sudo nano /opt/TOMCAT_[#]/conf/server.xml
```

***change server port***

        <server port="8x05" shutdown="SHUTDOWN">


***change connector port***

        <Connector port="8x8x" protocol="HTTP/1.1"
                   connectionTimeout="20000"
	               redirectPort="8443" />
	               
***change redirect port***

        <Connector port="8x09" protocol="AJP/1.3" redirectPort="8443" />




* Create script to run Tomcat as a service (repeat for each instance)
```sh
	$ sudo touch /etc/init.d/TOMCAT_[#]
```

* Edit script (repeat for each instance)
```sh
	$ sudo nano /etc/init.d/TOMCAT_[#]	
```

* Copy and paste this script (adding your changes). The script end is "exit 0" row. The script is not complete and it has to be update. Edited from [here](http://wiki.openkm.com/index.php/Configure_Tomcat_service_linux "script tomcat")



```sh

#!/bin/bash
#
# tomcat7qrcode     This shell script takes care of starting and stopping Tomcat
#
# chkconfig: - 80 20
#
### BEGIN INIT INFO
# Provides: tomcat8_qrcode
# Required-Start: $network $syslog
# Required-Stop: $network $syslog
# Default-Start:2 3 4 5
# Default-Stop: 0 1 6
# Description: Release implementation for Servlet 2.5 and JSP 2.1
# Short-Description: start and stop tomcat_qrcode
### END INIT INFO

## Source function library.
#. /etc/rc.d/init.d/functions
NAME="$(basename $0)"
export JAVA_HOME=/usr/lib/jvm/jre
export JAVA_OPTS="-Dfile.encoding=UTF-8 \
  -Dcatalina.logbase=/var/log/tomcat7qrcode\
  -Dnet.sf.ehcache.skipUpdateCheck=true \
  -XX:+DoEscapeAnalysis \
  -XX:+UseConcMarkSweepGC \
  -XX:+CMSClassUnloadingEnabled \
  -XX:+UseParNewGC \
  -XX:MaxPermSize=128m \
  -Xms512m -Xmx512m"
export PATH=$JAVA_HOME/bin:$PATH
# Change the below value!!!
TOMCAT_HOME=/usr/share/tomcat8_qrcode
SHUTDOWN_WAIT=20


tomcat_qrcode_pid() {
	echo `ps aux | grep -w $TOMCAT_HOME | grep -v grep | awk '{ print $2 }'`
}

start() {
  pid=$(tomcat_qrcode_pid)
  if [ -n "$pid" ]
  then
    echo "${NAME} is already running (pid: $pid)"
  else
    # Start tomcat qrcode
    echo "Starting ${NAME}"
    ulimit -n 100000
    umask 007
    sudo $TOMCAT_HOME/bin/startup.sh
  fi


  return 0
}
stop() {
  pid=$(tomcat_qrcode_pid)
  if [ -n "$pid" ]
  then
    echo "Stoping ${NAME}"
    sudo $TOMCAT_HOME/bin/shutdown.sh

    let kwait=$SHUTDOWN_WAIT
    count=0;
    until [ `ps -p $pid | grep -c $pid` = '0' ] || [ $count -gt $kwait ]
    do
      echo -n -e "\nwaiting for processes to exit";
      sleep 1
      let count=$count+1;
    done

    if [ $count -gt $kwait ]; then
      echo -n -e "\nkilling processes which didn't stop after $SHUTDOWN_WAIT seconds"
      kill -9 $pid
    fi
  else
    echo "${NAME} is not running"
  fi

  return 0
}

case $1 in
start)
  start
;;
stop)
  stop
;;
restart)
  stop
  start
;;
status)
  pid=$(tomcat_qrcode_pid)
  if [ -n "$pid" ]
  then
    echo "${NAME} is running with pid: $pid"
  else
    echo "${NAME} is not running"
  fi
;;
esac
exit 0


  ```
  
* Make the script executable (repeat for each instance) 
```sh
	$ sudo chmod 755 /etc/init.d/TOMCAT_[#]
```
* Start tomcat service (repeat for each instance)
	```sh
	$ sudo service TOMCAT_[#] start
    ```
    
* Add tomcat service in startup (repeat for each instance)
	```sh
	$ sudo chkconfig TOMCAT_[#] on
    ```



#### AWS console

* Select your machine instance  
* Under DESCRIPTION tab, in security gorups option, lunch *"lunch-wizard-xx"*   
* In the new console right click on your groupID (create new one if you haven't made one yet)  
* Click on "Edit inbound rules"  
* Open the port of your instance (repeat for each port you need). They are the "connector port" in server.xml file  

```sh
	    TYPE		    PROTOCOL	    PORT RANGE 	    SOURCE 
    custom TCP Rule       TCP             8x8x         Anywhere
    custom TCP Rule       TCP             8x8x         Anywhere
```

* Save changes  
* Start your machine



## CMD (as administrator)

* Connect to AWS using ssh protocol (!!!!!! It might be changed if your are not using Elastic IP)  
```sh
    $ ssh -v -i [PATH_KEY]\KEY_SSH.pem ec2-user@AWS_SERVER_IP
```

* Check if your TOMCAT_[#] instances are running
```sh
    $ sudo service --status-all | grep -i "TOMCAT_"
```



## Browser

* Open a browser and check your tomcat instance   
```sh
[AWS_SERVER_IP]:8x8x
```
