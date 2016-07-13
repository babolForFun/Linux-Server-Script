
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
        $ scp -i "[PATH_KEY]\KEY_SSH.pem" "[PATH_TOMCAT]\apache-tomcat-7.0.70.tar.gz" ec2-user@[AWS_SERVER_IP]: 
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
#!/bin/sh
 
### BEGIN INIT INFO  
# Provides:          tomcat  
# Required-Start:    $remote_fs $syslog  
# Required-Stop:     $remote_fs $syslog  
# Default-Start:     2 3 4 5  
# Default-Stop:      0 1 6  
# Short-Description: Start and stop Apache Tomcat  
# Description:       Enable Apache Tomcat service provided by daemon.  
### END INIT INFO
 
ECHO=/bin/echo  
TEST=/usr/bin/test  
TOMCAT_USER=root                                         # YOUR USER  
TOMCAT_HOME=/opt/TOMCAT_[#]                              # HOME OF YOUR TOMCAT INSTANCE  
TOMCAT_START_SCRIPT=$TOMCAT_HOME/bin/startup.sh  
TOMCAT_STOP_SCRIPT=$TOMCAT_HOME/bin/shutdown.sh  
 
$TEST -x $TOMCAT_START_SCRIPT || exit 0  
$TEST -x $TOMCAT_STOP_SCRIPT || exit 0  
 
start() {  
    $ECHO -n "Starting TOMCAT_[#]"                                  # TOMCAT SERVICE NAME  
    su - $TOMCAT_USER -c "$TOMCAT_START_SCRIPT &"  
    $ECHO "."  
}  
   
stop() {  
    $ECHO -n "Stopping TOMCAT_[#]"                                  # TOMCAT SERVICE NAME  
    su - $TOMCAT_USER -c "$TOMCAT_STOP_SCRIPT 60 -force &"  
    while [ "$(ps -fu $TOMCAT_USER | grep java | grep tomcat | wc -l)" -gt "0" ]; do  
        sleep 5; $ECHO -n "."  
    done  
    $ECHO "."  
}  
   
case "$1" in  
    start)  
        start  
        ;;  
    stop)  
        stop  
        ;;  
    restart)  
        stop  
        sleep 30  
        start  
        ;;  
    *)  
        $ECHO "Usage: TOMCAT_[#] {start|stop|restart}"              # TOMCAT SERVICE NAME  
        exit 1  
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
