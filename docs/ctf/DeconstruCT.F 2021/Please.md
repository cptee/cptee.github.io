---
layout: default
title: Please
parent: DeconstruCT.F 2021
grand_parent: Capture the Flag
nav_order: 1
has_children: false
permalink: /ctf/deconstructf2021/please
---

# Please Challenge Writeup (03/10/2021)
As the name suggests, this challenge requires you to adjust your requests according to the server in order to retrieve the flag. 

## 1. Examine the target
![Screenshot 1](/assets/images/deconstructf/please1.png)

The target consists of a page asking you to provide an username.
Let's try a random one (cptee):

![Screenshot 2](/assets/images/deconstructf/please2.png)

It seems that Clancy is the only username that can access the website.

## 2. Submit Clancy as the Username
When we submit Clancy as the username, a different error is shown:

![Screenshot 3](/assets/images/deconstructf/please3.png)


That's is odd. Let's try analyzing our request with <b>Burp Suite</b>.

![Screenshot 4](/assets/images/deconstructf/please4.png)

## 3. Admin_Access Flag

There is a flag called <b>Admin_Access</b> set as False.
Switch it to True and the following happens:

![Screenshot 5](/assets/images/deconstructf/please5.png)

Seems like we have the wrong browser.
Refresh the page and send the request to Burp's Repeater.

## 4. User-Agent
Since the server will only accept requests from DeemaBrowser, by changing our User-Agent we can bypass this requirement.

![Screenshot 6](/assets/images/deconstructf/please6.png)

Response: 

![Screenshot 7](/assets/images/deconstructf/please7.png)

## 5. Basic Authentication
We have to provide an Authorization Header with "What'sTheMagicWord?" as the passphrase. Please add the header below to your request:

```
Authorization: Basic <credentials>
```

Substitute credentials to What'sTheMagicWord? base64 encoded

```
Authorization: Basic V2hhdCdzVGhlTWFnaWNXb3JkPw==
```

Response:

![Screenshot 8](/assets/images/deconstructf/please8.png)

Seems like we also need to provide a date in order to access the files.

## 6. Date
Syntax
```
Date: <day-name>, <day> <month> <year> <hour>:<minute>:<second> GMT
```

Let's provide a date in April 2021 and analyze the response:
```
Date: Thu, 1 Apr 2021 12:00:00 GMT
```

![Screenshot 9](/assets/images/deconstructf/please9.png)

Change the date to Monday, 5th of April:
```
Date: Mon, 5 Apr 2021 12:00:00 GMT
```

![Screenshot 10](/assets/images/deconstructf/please10.png)

Retrieve the Flag and that's it!

Thank you for reading.













