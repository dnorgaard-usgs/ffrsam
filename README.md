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

