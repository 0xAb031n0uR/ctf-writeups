# B!inded! (Web Challenge)

## Description 

```
No fuzzing is needed, just get the flag , Format : EGCERT{flag}
Points : 200

```

## Enumeration 
 
once i opened the challenge i found this page 

[loginpage](login-page-photo)

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
then i typed ```aboelnour" or 1=1 -- -``` and boom i have got response ```Wrong Password```


