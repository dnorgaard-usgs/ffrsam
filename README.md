# ffRSAM

ffRSAM is a web-based application to allow users to view plots of pre-calculated Real-time Seismic-Amplitude Measurement (RSAM) and frequency filtered (ff) RSAM.  The system has a back-end Python script run by cron jobs to retrieve trace data from a wave server (e.g. Winston), calculate RSAM and ff RSAM based on pre-determined frequency bands, and store the calculated values in a MariaDB database.  The front-end, or user interface, is a web-application written in plain HTML and JavaScript. REST API written in PHP is used to query the database.

## System Requirements
Python 3

PHP 7

MySQL 5

The ffRSAM application has been developed and tested on CentOS 7 Linux platform and the instructions here are written for this operating system.  However, the programming languages used are platform independent and should run on any operating system with minor modifications to the instructions.  The instructions also assume installation of the Python scripts, MariaDB database, and Apache web server will be on the same server, though they are not required to be.  

## Installation Instructions

Installation consists of 3 parts:
1.  Database install
2.  Python scripts install
3.  Web application install

### Database Install
1. Install MariaDB (or MySQL) if it is not already installed.
```
yum install mariadb-server
systemctl enable mariadb
systemctl start mariadb
mysqladmin -u root password 'newpassword' 
```

2. Create rsam database and objects. The rsam.sql file is in the sql folder.

```
mysql -u root -p < rsam.sql
```
Following tables will be created:

**channels**

This table stores a summary of what data RSAM values are calculated for.  It stores the SCNL channel name, the period (in seconds), start time, end time, low pass value of frequency band, and high pass value of frequency band. The start time and end times denote when the RSAM values for this channel period and band was first and last calculated.

**rsam**

This table stores the actual rsam values.

3. Create user for accessing rsam database. Replace ‘password’ with rsamuser password:

```
mysql -u root -p
mysql> CREATE USER 'rsamuser'@'localhost' IDENTIFIED BY 'password'; 
mysql> GRANT ALL PRIVILEGES ON *.* TO 'rsamuser'@'localhost' WITH GRANT OPTION;
mysql> flush privileges;
```

### Python Scripts Install

Instructions below assumes use of Anaconda, which is recommended.  
1)	Install Python 3: https://docs.anaconda.com/anaconda/install/linux/. The instructions will assume you have installed anaconda3 under /opt.  If you install anaconda3 elsewhere, replace /opt/anaconda3 in this instruction with your installation directory.  
2)	Install ObsPy: https://github.com/obspy/obspy/wiki/Installation-via-Anaconda. Instructions here creates an ‘obspy’ environment.  Use ‘rsam’ here instead, e.g. conda create -n rsam python=3.7.  
3)	Install ffRSAM Python scripts by unzipping rsam.zip to a directory on your server.  
4)	Configure ffRSAM Python scripts:  
a)	Verify PYTHON_HOME in run.sh uses the correct anaconda installation and environment selected in steps 1 & 2.  If not, edit to use correct path.  
b)	Copy config_template.py to config.py and edit config.py (watch out for proper placement of brackets and commas):  
*buffer* – This variable specifies the potential latency, i.e. time between when seismic trace data is obtained at the station to when the data reaches the wave server.  Larger value here will help accommodate potential delays in the data acquisition process.  Default value is 20 minutes. 
*db* –Stores the rsam database host, username, and password.  
*wave_server* – Multiple wave server information can be stored here.  Example configuration is provided but should be replaced with the information (name, host, port) of the actual wave server where the trace data is stored.  
*default_bands* – These are the default bands that will be used to calculate the ff RSAM data. It is not recommended to change this value.  Default bands are [[0.1,1],[1,3],[1,5],[1,10],[5,10],[10,15],[15,20]].  
*channels* – The channels for which the rsam will be calculated are specified here.  Examples are provided but should be replaced with channels in your wave server.  RSAM calculations are typically performed on the vertical (Z) component of a station.  
5)	Test the scripts installation by running “./run.sh 10”.  The run.sh script takes in the period, in minutes, as the argument.  It will calculate the RSAM and ff RSAM values for the channels specified in config.py for the specified period.  End time for the RSAM is calculated as [ current time ] – [ buffer ] – ([ current time]%[period]).  Resolve any errors and ensure that the 10 minute RSAM data is stored in the database.  
6)	Set up cron jobs. The parameter to run.sh is the period in minutes. The desired periods to use for RSAM calculations may vary by installation.  Below example sets up cron jobs to calculate RSAM/ffRSAM for 10 minutes, 1 hour, 4 hours, 12 hours, and 1 day.  Replace ~/scripts with the directory where rsam.zip was unpacked.  Replace /data/log with the directory where you want the log files to go.  In this set up only logs for the latest run of each job is stored. For periods of hour or more, specify the minute time as the amount of buffer or lag time used in config.py.  For example, config_template.py has a default buffer of 5 minutes.  The cron jobs below reflect that 5 minutes in the minute configuration (last 4 entries).

```
*/10 * * * * ~/scripts/rsam/run.sh 10 > /data/log/rsam10.log 2>&1
5 * * * * ~/scripts/rsam/run.sh 60 > /data/log/rsam60.log 2>&1
5 0,4,8,12,16,20 * * * ~/scripts/rsam/run.sh 240 > /data/log/rsam240.log 2>&1
5 0,12 * * * ~/scripts/rsam/run.sh 720 > /data/log/rsam720.log 2>&1
5 0 * * * ~/scripts/rsam/run.sh 1440 > /data/log/rsam1440.log 2>&1

```
Ensure each cron job specified is running without errors and corresponding results are stored in the database.

### Web Application Install

1)	Install Apache web server:
```
yum install httpd
systemctl start httpd
systemctl enable httpd
```

2)	Install PHP and required libraries:
```
yum install php
yum install php-common
yum install php-mysqli
yum install php-xmlrpc
yum install php-json
yum install php-gd
```

3)	Copy the web directory to /var/www/html/rsam.
`cp -r web /var/www/html/rsam`
4)	View the file api/v1/api.php in the rsam directory.
a)	By default, it is expected that the database credentials are stored in /var/www/external/mysql.php which defines $db_server, $db_user, and $db_password.  If you do not have this file, create it as root, and then change ownership to root:apache with permissions set to 640 to ensure only root and the apache user can view the database credentials.
```
cd /var/www/external
vi mysql.php
```

Example content of mysql.php:
```
<?php
  $db_server="127.0.0.1";
  $db_user="rsamuser";
  $db_password="rsampw";
?>
```
Set permissions:
```
chown root:apache mysql.php
chmod 640 mysql.php
```


Once done ensure the web application works.  Go to the rsam web application url.  Example: http://localhost/rsam/.  The web application will look like below (channels, period, and frequency filters may differ).


