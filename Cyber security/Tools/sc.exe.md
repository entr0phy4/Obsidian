SC is a command line program used for communicating with the
Service Control Manager and services.

---

```powershell
sc.exe create <NEW_SERVICE_NAME> binPath="example.exe" 
```
Create a service.

---

```powershell
sc.exe config ServiceToChange binPath="new_config"
```
Change config of existent service.