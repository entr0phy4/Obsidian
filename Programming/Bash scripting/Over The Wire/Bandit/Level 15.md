**Password**: 8xCjnmgoKbGLhHFAZlGE5Tmu4M2tKJQo
**Tools**: [[openssl]]

## Solution
The password for the [[Level 16]] can be retrieved by submitting the password of the current level to **port 30001 on localhost** using SSL/TLS encryption.

we can use [[openssl]] to establish the connection encrypted.
```bash
openssl s_client --connect 127.0.0.1:30001
```

when we connect succesful, send the current password.