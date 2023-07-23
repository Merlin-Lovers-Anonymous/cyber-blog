---
title: "Java Code Analysis!?! PicoCTF 2023 Web Writeup"
date: 2023-07-22
description: "A writeup for the Java Code Analysis challenge in PicoCTF 2023."
tags: ["PicoCTF", "Writeup", "Web"]
type: post
weight: 20
showTableOfContents: false
---

# Description
BookShelf Pico, my premium online book-reading service. I believe that my website is super secure. I challenge you to prove me wrong by reading the 'Flag' book! Here are the credentials to get you started:

Username: "user"

Password: "user"

# Solution
Before even looking at the source code let's look through the website and see what we have to work with. Logging in with the provided credentials presents us with this page:

![website](/images/Java-Code-Analysis-PicoCTF-2023-Web-Writeup/website.png)

On the top right we see our current account’s privilege of “Free”. We also see the “FLAG” book which has the admin tag. Presumably we must trick the website into thinking we are an admin, and thus letting us read the flag. After trying to read the flag book and analyzing the traffic we see the following:

![network](/images/Java-Code-Analysis-PicoCTF-2023-Web-Writeup/network.png)

In the request headers we see there is a “Bearer” authorization header followed by some token. In storage we can also see a cookie with a json and token:

![cookie](/images/Java-Code-Analysis-PicoCTF-2023-Web-Writeup/cookie.png)

Searching “bearer” in the source files provided to us we find this code:

![jwtService](/images/Java-Code-Analysis-PicoCTF-2023-Web-Writeup/jwtService.png)

From this we can see that the website is using “jwtService” which after some googling is revealed to be an encoded json token. In the JwtService class we see something intriguing:

![secretKey](/images/Java-Code-Analysis-PicoCTF-2023-Web-Writeup/secretKey.png)

The secret key being used to sign the jwt tokens is being created by a different method called getServerSecret() from a different class. Looking at the getServerSecret() implementation we might be initially overwhelmed by all of the code, however upon further inspection the secret key is not so secret:

![generateRandomString](/images/Java-Code-Analysis-PicoCTF-2023-Web-Writeup/generateRandomString.png)

As seen in the picture the getServerSecret() method returns newSecret, which is set to the return of the method called generateRandomString(32). Looking at that method we see that it always returns “1234”. Boom, we now have the secret key used to sign the jwt token. Now we are ready to forge our token. Taking the token we found earlier and putting it into cyberchef we can decode it into the json: 

![jwtDecode](/images/Java-Code-Analysis-PicoCTF-2023-Web-Writeup/jwtDecode.png)

Then we can copy our decoded json and simply change our role to “Admin” and put our userId to “2”, then sign it with our secret key:

![jwtEncode](/images/Java-Code-Analysis-PicoCTF-2023-Web-Writeup/jwtEncode.png)

Then we can copy our json and put it in the cookie we saw earlier as the “token-payload” and then put our encoded token into the “auth-token” field:

![cookieManipulation](/images/Java-Code-Analysis-PicoCTF-2023-Web-Writeup/cookieManipulation.png)

After this simply refreshing the page will fool the page into thinking we are an admin and allowing us to see the flag:

![flag](/images/Java-Code-Analysis-PicoCTF-2023-Web-Writeup/flag.png)

Flag: picoCTF{w34k_jwt_n0t_g00d_ca4d9701}