---
layout: post
title: "Securing PHP-Flash Communcation"
date: 2012-01-19T00:00:00-05:00
---

Virtually every computer in the world has flash installed. Whether you're running Debain or Windows 7, there's a distribution of Adobe Flash available for you. That makes Flash the most interoperable environment available. Now let's say you want to embed Flash into a website. Either you want a submission form, chat box, or a simple game, you will most likely need to communicate back and forth with the server. Doesn't sound hard at all, especially with so many libraries available and so many services you can use(<a href="http://en.wikipedia.org/wiki/Representational_State_Transfer">REST</a>, <a href="http://en.wikipedia.org/wiki/XML-RPC">XMLRPC</a>, <a href="http://en.wikipedia.org/wiki/JSON">JSON</a>,  <a href="http://www.gotoandplay.it/_articles/2003/12/xmlSocket.php">XMLSocket </a>for low latency communication, or just a simple HTTP GET request to the PHP page). However, how sure are you of the validity of the data transferred from the client-side Flash app back to your server?

All programming languages preach <strong>to never trust user input directly</strong>. For a simple example, a good program will even check that a person's name is made of valid and accepted characters. However, too many people unfortunately ignore this rule when they create Flash apps. Even if you're creating a contact form field, there are many ways a user could exploit your site if you don't properly secure the client's input. Most notably, he could create an <a href="http://en.wikipedia.org/wiki/SQL_injection">SQL injection</a> the input fields. 

Fortunately for form fields, there are bullet proof ways to secure them. In the following section I will discuss those. However, I will also talk about more sophisticated applications such as games which track levels and submit high scores. In these cases, it's not possible to secure your high score list 100% , but you can make hard enough for virtually everyone to back down. 

<strong>Securing simple form fields: </strong>

The first basic step is to obviously validate the form fields inside of Flash. But this is by no means makes your application secure. If anything, it saves an extra HTTP request. You will also have to duplicate this form verification on the server side. The most important thing you can do on server-side verification is check for an SQL injection. If you're using MySQL, pass it through the <a href="http://php.net/manual/en/function.mysql-real-escape-string.php">mysql_real_escape_string</a> or <a href="http://www.php.net/manual/en/mysqli.real-escape-string.php">mysqli_real_escape_string</a>. I strongly prefer you use mysqli_real_escape_string because it avoids some security flaws present in the former function. 

<pre lang="cpp-qt" lineno="1">
<?php
$mysqli = new mysqli('root','pass','db');
$username = isset($_POST['username'])?$mysqli->real_escape_string($_POST['username']):'';
?>
</pre>

Now that we secured basic input, we need to protect against spoofing and DDOS attacks. Suppose that an attacker wants to slow your site down considerably, or even bring it down. One way he could do it is by flooding the server with requests. If the server believes this request came from your trusted application, it will send it to your database. Even though your database will filter the content, hundreds of thousands of requests to your database will most likely bring it down. One way to protect against spoofing is by using tokens. 

<strong>Tokens</strong>: 

Here's one way to approach it. Every time your trusted form sends a request to the server, it will also send along a secure token. 

First the server will send a random string to the Flash and save the encrypted version of this string in the database. This string will be encrypted using a token that will be hard-coded into the Flash game as well as in the PHP. Now, whenever the Flash form loads, it must request a random string to be sent to it. It will then use the secure key that you hard coded to create a token and send the token along with the response. Once the server recieves your request, it will compare your encrypted token with the ones stored in the database. If there is a match, only then should you process the form. Also remember to delete the matching token from the database after that. If the token received does not match any token that have been generated in say, the last hour, then discard that request.

<strong>Encrypting Your Source</strong>

The last part is to obfuscate your action-script source good enough to deter hackers from trying to find the form fields and even the encryption key. Encrypt the entire action-script with a library like <a href="http://code.google.com/p/as3corelib/">as3corelib</a>. This will prevent the vast majority of people from proceeding. They will not be able to read your source and even think about finding the encryption key.  Now whenever a user submits the form from your trusted app, it will also send the token. However, decompiling action-script is much easier then other languages because action-script is a very high level programming language, not to mention the byte-codes are well documented. Here are some ways to deter or confuse an attacker who has decrypted your application:

	<li> Replacing key byte arrays with functions that calculate the key</li>
	<li>Placing numerous fake encrption keys throughout the app.</li>

I also recommend this strategy for securing Flash games, so I will not go over it again. 

<strong>Securing Flash Game Points Submission</strong>

The biggest problem with Flash-based games is how easily the high score lists can be hacked. One of the best(and most complicated) ways to protect against this is to put all the game logic on the server. That way, whenever a user submits a new high score, your server side script checks if that score is possible with the moves that the user has. For example, if the highest recorded level that the user has reached is 5, there is no way for him to have reached score x. For this to work, you cannot send HTTP requests indicating when the level has changed. The change must be determined on the server, after the Flash client has sent some command. This command should trigger a change in the level. 

If you have a fast paced game, where there are a lot of command getting executed at the same time, you could <a href="http://help.adobe.com/en_US/ActionScript/3.0_ProgrammingAS3/WS5b3ccc516d4fbf351e63e3d118a9b90204-7c60.html">open a socket.</a> 

There are many other ways to secure Flash apps, but to have basic security, I advise that you follow the encryption method mentioned in this post. 