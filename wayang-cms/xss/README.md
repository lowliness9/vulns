# wayang-cms Cross-site scripting vulnerability
---
### wayang-cms official download address
Github official download，[wayang-cms](https://github.com/ketutd/wayang-cms)
---
### Build a recurring environment
Download and install phpStudy, download wayang-cms and copy all files inside to phpStudy's web directory, connect to mysql and create a new database wayang-cms, start Apache to access `http://127.0.0.1/wy_install/` set up installation. After the installation is complete, import cms.sql in wy_install into the database, and the successful operation is as follows
![](https://lowliness9.github.io/post-images/1605356351107.png)
---
### Vulnerability analysis
The vulnerability occurs on line 316 of index.php
![](https://lowliness9.github.io/post-images/1605325310667.png)
Through debugging, it is found that `$datamod['widget_module']` value is visitor, visit the homepage to view the source code
![](https://lowliness9.github.io/post-images/1605325452724.png)
![](https://lowliness9.github.io/post-images/1605325415653.png)
The file included here is `wy_controlls/wy_side_visitor.php`, locate the line 49 of `wy_controlls/wy_side_visitor.php`, the code is as follows:
![](https://lowliness9.github.io/post-images/1605325530824.png)
The code directly takes the value from `$_SESSION['visitor']['visitorip']` and locates it to the 11th line.
![](https://lowliness9.github.io/post-images/1605325637644.png)
`$_SESSION['visitor']` is assigned as an array, `$_SESSION['visitor']['visitorip']` is the visitor IP, its calling method is ip(), locate this method, the file is `wy_get_ipaddress .php`.
![](https://lowliness9.github.io/post-images/1605325786777.png)
Here, the source IP can be forged by `X-Forwarded-For`, this method returns directly without any processing of the data, and then assigns it to `$_SESSION['visitor']`, and finally outputs directly to form a reflective XSS.
PayLoad: `X-Forwarded-For: <script>alert('xss')</script>`
---
### Vulnerability proof
![](https://lowliness9.github.io/post-images/1605325944155.png)
![](https://lowliness9.github.io/post-images/1605325868585.png)
---
