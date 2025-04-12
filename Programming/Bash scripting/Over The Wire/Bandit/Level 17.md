**Password**: EReVavePLFHtFlFsjn3hyzMlvSuSAcRD
**Tools**: [[diff]]

## Solution 
There are 2 files in the homedirectory: passwords.old and passwords.new. The password for the [[Level 18]] is in passwords.new and is the only line that has been changed between passwords.old and passwords.new

We can use [[diff]] to see only the lines changes between two files.
```bash
diff password.new password.old
```