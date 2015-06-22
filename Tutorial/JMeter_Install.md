# (Tutorial) JMeter Install and How to use

## Install the Apache JMeter

**Point of this course**

* Install JMeter   (Windows7)
* Install Java jre (Windows7)
* How to Use JMeter (a little)

**環境情報**

- [JMeter PC] 10.0.2.4
- [Application Server] 10.0.2.8

## Download and Install the Apache JMeter 

Download site:
[Apache JMeter](http://jmeter.apache.org/download_jmeter.cgi)

unpack the zip and deploy anywhere:
[apache-jmeter-2.13.zip](http://ftp.jaist.ac.jp/pub/apache//jmeter/binaries/apache-jmeter-2.13.zip)

## Download and Install the Java jre

Download site:
[Java jre 8](https://java.com/ja/download/manual.jsp)

Install [jre-8u45-windows-x64.exe] or [jre-8u45-windows-i586.exe]  
(double click and you can change the install directory.)

## How to Use (Stress Test)

(on the windows explore)

1. open the directory [xxx/apache-jmeter-2.13/bin]  
1. looking for the [jmeter.bat]  
1. double click it and start the GUI menu  
1. add [Threads(Users)]-[Threads Group], the first  
  set some value:   
    - threads count    : 1000  
    -  Ramp-Up Term(s) : 10  
1. add [Sampler]-[http request], the second  
  set some value:  
    - web server ip : 10.0.2.8  
    - port number   : 8000  
1. add [Listener]-[show the result by the table], the third  
1. Run the [Run] buttom and you can see the result  
