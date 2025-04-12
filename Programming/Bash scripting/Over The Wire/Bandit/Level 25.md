**Password**: iCi86ttT4KSNe1armKiwbQNmB3YJP3q4
**Tools**: [[more]], [[ssh]]

**Prev**: [[Level 24]]
**Next**: [[Level 26]]
## Solution
Logging in to bandit26 from bandit25 should be fairly easy… The shell for user bandit26 is not **/bin/bash**, but something else. Find out what it is, how it works and how to break out of it.

> NOTE: if you’re a Windows user and typically use Powershell to `ssh` into bandit: Powershell is known to cause issues with the intended solution to this level. You should use command prompt instead.

when we try use `bandit26.sshkey` to connect via [[ssh]] to `bandit26` we found the session is automatically closed. so being `bandit25` we can search whats is the shell for `bandit26`.
![[Pasted image 20250410131521.png]]
and read it.
![[Pasted image 20250410131547.png]]
we see the shell of `bandit26` is spawn a text using the tool [[more]], we can approach that with a `v` mode of more edit another file. That just works if message is small enough.
![[Pasted image 20250410131914.png]]There we not see `more` is executed but if we reduce the size
![[Pasted image 20250410132101.png]]
`more` is executed.
now if we press the key `v` and type `:e /etc/bandit_pass/bandit26` to try edit the password of `bandit26` 
![[Pasted image 20250410132545.png]]
we can see the password. but we need spawn a shell.
with `more` we can declare variables, so let's try this:
![[Pasted image 20250410133410.png]]
we have an variable called `shell` equals `/bin/bash`, if we try call that variable.
![[Pasted image 20250410133512.png]]
we got a shell.