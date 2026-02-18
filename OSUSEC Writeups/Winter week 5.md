This week was centered around Git, the universal coding tool that keeps track of projects with version history, collaboration, and much more. Git stores every file ever used in the project, and still managed by it, so if access & files are not controlled carefully this could lead to exposed files, private keys, etc.

# Flag 1:

### Initial git repo exploration
The story here is that one of our clients got pwned by EvilCorp recently, and a few of their files got encrypted by ransomware (our flag is in there!), the payment website looks as follows:
![[Pasted image 20260209170151.png]]

This is quite scary and all, but they left their .git repository exposed (as was implied by the prior lecture on git). By navigating to website/.git/HEAD we can get the name of the repository used by the website, and we can start looking into the rabbithole:

![[Pasted image 20260209170556.png]]

looks like the primary repo is master, so lets take a look at master, starting with the ref link:

![[Pasted image 20260209170827.png]]

We get a long hash, that doesn't really mean much unless you know that git stores file, directory, and snapshot data in these hashed files - this is likely a snapshot that could tell us about the repository.

Navigating to .git/objects/first_two_chars_of_hash/rest_of_hash (not the literal filepath), downloads that object file, which we can throw into an empty git repo and use git cat-file -p to read:

![[Pasted image 20260209171139.png]]

Looks like it's got a a file tree, which we could definitely extract but I want to get smarter before I continue. At the bottom, we see the note that reads "Add warnings to README", README's are stored in the base project directory, so let's see if we can extract that:

![[Pasted image 20260209171318.png]]

Sure can! and it looks like there's some sort of private key encryption with this code.

### Decryption code
Downloading the provided flag and key files we see these two bits of data:

![[Pasted image 20260209171547.png]]

The key and flag files are clearly encrypted, no doubt, but the flag looks almost identical to the type of encryption we saw in last week's Fernet encrypted data - this is made clearer when looking at the decryption code that literally imports Fernet:

![[Pasted image 20260209171945.png]]

It looks like the code has two modes, decryption and key file - we could try to bypass the private key check in the code but we can't decrypt the key without it. What we do have, in the README file, is the implication that there were worries about the secret key getting leaked into the commits

### Grabbing our flag

.git/logs/master doesn't exist, but .git/logs/HEAD does, which shows us this beautifully simple version history:

![[Pasted image 20260209172232.png]]

Clearly the private key was included in one of the commits - lets grab it:

First we navigate to the hash of the commit before the one that talks about the env file, and we grab its object file:

![[Pasted image 20260209172514.png]]

Looks right, lets keep digging:

![[Pasted image 20260209172703.png]]

There's that env file! grab that and we're golden

![[Pasted image 20260209172829.png]]

Now that we have the private key, decrypting the key, and then extracting the flag should be no biggie:

![[Pasted image 20260209173252.png]]

I threw the private key into a .env file just like the hackers set up, and with that I was able to run decrypt on the key file, then on the flag file, getting us out data back! (also the flag, mostly that)