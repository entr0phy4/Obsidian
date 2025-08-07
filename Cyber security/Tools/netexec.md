Description...

---

```bash
netexec smb <IP_ADDRESS>
```
Enumerate [[Server message Block (SMB)]].

---

```bash
netexec -L <IP_ADDRESS> -N
```
List shares resources using a null session. (No user, no password)

---

```bash
netexec smb <IP_ADDRESS> -u 'username' -p 'password'
```
Validate credentials.

---

```bash
netexec winrm -i <IP_ADDRESS> -u 'username' -p 'pasword'
```