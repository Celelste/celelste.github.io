This week's challenge was all about encryption - We were given a large chunk of (likely) python code to start:

![[Pasted image 20260202144321.png]]

We were also given the "program output", whose importance will be relevant for the second flag:

```
Program output: 1769477068.8361542 67414141414142706542504d4e597439356876444265624e356d77727974597663724a5145666c70475f7261316372616d63392d59312d4735576442646b6c6d48315a4d7474794b316f77484a70315064486c6d4b59386a5f764c75366b6131336a57443139375741485249674964355a7347524a62343d
```

from this, there are a series of "eval" code blocks, which when all pulled out gives:

```
from cryptography.fernet import Fernet
from ast import literal_eval
import hashlib
import struct
import base64 as bs64
import random
import time
import hmac

htp = lambda s,c,d=6:(lambda h:str((struct.unpack('>I',h[(h[-1]&15):(h[-1]&15)+4])[0]&2147483647)%10**d).zfill(d))(hmac.new(s,struct.pack('>Q',c),hashlib.sha1).digest())

tpt = lambda s,t=30,d=6:htp(bs64.b32decode(s.upper()+'='*((8-len(s)%8)%8)),int(time.time()//t),d)

ctpttk=lambda t:(lambda b:bs64.urlsafe_b64encode(b).decode().encode())(random.getrandbits(256).to_bytes(32,'big') if not random.seed(t) else None)

rfd=lambda p:open(p,'rb').read()

e=lambda p:(lambda k,o,f,d:print(f.encrypt(d).hex()))("J5JVK6ZQNZSV6VDJNVSV6UDBMRZV6ZTJNRWGK4T5",(o:=tpt("J5JVK6ZQNZSV6VDJNVSV6UDBMRZV6ZTJNRWGK4T5")),Fernet(ctpttk(o)),rfd(p))

print(time.time())

e("flag.txt")
```
(with minor edits for clarity)

# Flag 1

This whole thing is a mess, and I spent a long time trying to pick it apart (since I had never worked with it). After a while of analyzing the code, I got this rough code to run:

```
s="J5JVK6ZQNZSV6VDJNVSV6UDBMRZV6ZTJNRWGK4T5"

t=30

d=6

supper = s.upper(); print("supper", supper)

equals = ((8-len(s)%8)%8); print("equals", equals)

supperplus=supper+'='*equals; print("supperplus", supperplus)

decoded_supperplus = bs64.b32decode(supperplus); print("decoded_supperplus", decoded_supperplus)
```
(with the classic imports above)

From best what I can tell, this takes the string above, adds some amount of equal signs to it (which I later was told was for padding), then does a 32-bit decoding of it which spits out the first flag (along with a bunch of other stuff):

![[Pasted image 20260202164658.png]]

In fact, just decoding it from base32 gives us the first flag.

# Flag 2
With my initial success, I kept moving forward with my extraction of the code, but I seemingly got nowhere:

```
from cryptography.fernet import Fernet

from ast import literal_eval

import hashlib

import struct

import base64 as bs64

import random

import time

import hmac

  

timestamp=1769477068.8361542

data="67414141414142706542504d4e597439356876444265624e356d77727974597663724a5145666c70475f7261316372616d63392d59312d4735576442646b6c6d48315a4d7474794b316f77484a70315064486c6d4b59386a5f764c75366b6131336a57443139375741485249674964355a7347524a62343d"

  

s="J5JVK6ZQNZSV6VDJNVSV6UDBMRZV6ZTJNRWGK4T5"

t=30

d=6

supper = s.upper(); print("supper", supper)

equals = ((8-len(s)%8)%8); print("equals", equals)

supperplus=supper+'='*equals; print("supperplus", supperplus)

decoded_supperplus = bs64.b32decode(supperplus); print("decoded_supperplus", decoded_supperplus)

  

s=bs64.b32decodDe(supperplus)

  

c = int(timestamp//t)

  

struct_pack_c = struct.pack('>Q',c); print("struct_pack_c", struct_pack_c)

  

hmacx=hmac.new(s,struct_pack_c,hashlib.sha1); print("hmacx", hmacx)

  

h=hmacx.digest(); print("h", h)

  

htpbz=str((struct.unpack('>I',h[(h[-1]&15):(h[-1]&15)+4])[0]&2147483647)%10**d); print("htpbz", htpbz)

  

tpt=htpbz.zfill(d); print("htp", tpt)

  

b = random.getrandbits(256).to_bytes(32,'big') if not random.seed(t) else None; print("b", b)

  

ctpttkbdc=bs64.urlsafe_b64encode(b); print("ctpttk", ctpttkbdc)

ctpttkbec=ctpttkbdc.decode(); print("ctpttk", ctpttkbec)

ctpttk=ctpttkbec.encode(); print("ctpttk", ctpttk)

  

fern = Fernet(ctpttk); print("fern", fern)

  

read_file_data=open("flag.txt",'rb').read(); print("flag? ", open("flag.txt","r").read())
```

This spits out a bunch of stuff, none of which is the second flag!

![[Pasted image 20260202165020.png]]

At this point, my team had spent almost two hours trying to find the second flag here - our belief that the flag was in this code somewhere was based off of two assumptions:

1 - The first thing we noticed after running the program, was that a long number and a string were outputted to the terminal - we assumed that this was identical to the program output provided by the challenge details

2 - We already had all the information that we needed to solve the challenge

These were both wrong.

The first assumption can easily be identified as wrong when comparing the number in our output (which was given by time.time) to the one given by the challenge description:

```
1770080489.036953
```
our number

```
1769477068.8361542
```
their number

This detail would have clued us into the solution, but on the day of the event the first few numbers were identical, and so on first glance it looked the same

The second assumption was erased when we were told this was a timestamp, and when we found the solution, connected that the slides explaining how all this works were short, and not very informative

now knowing this, we have an algorithm that takes a timestamp and outputs a key, which can be used to encrypt a message - if you know that time you can decode the message, which my teammate did to get the second flag with this code:

```
import hmac

import hashlib

import struct

import base64

import random

import time

from cryptography.fernet import Fernet

  

# --- 1. The Inputs You Provided ---

hex_string = "67414141414142706542504d4e597439356876444265624e356d77727974597663724a5145666c70475f7261316372616d63392d59312d4735576442646b6c6d48315a4d7474794b316f77484a70315064486c6d4b59386a5f764c75366b6131336a57443139375741485249674964355a7347524a62343d"

target_timestamp = 1769477068.8361542

secret_base32 = "J5JVK6ZQNZSV6VDJNVSV6UDBMRZV6ZTJNRWGK4T5"

  

# --- 2. Re-implementing the Logic ---

  

def get_hotp_token(secret, intervals_no):

"""Refactored version of the 'htp' lambda."""

key = base64.b32decode(secret, casefold=True)

msg = struct.pack(">Q", intervals_no)

h = hmac.new(key, msg, hashlib.sha1).digest()

o = h[19] & 15

h = (struct.unpack(">I", h[o:o+4])[0] & 0x7fffffff) % 1000000

return str(h).zfill(6)

  

def get_encryption_key(seed_str):

"""Refactored version of the 'ctpttk' lambda."""

# This recreates the exact key generation flaw

random.seed(seed_str)

key_bits = random.getrandbits(256)

key_bytes = key_bits.to_bytes(32, 'big')

return base64.urlsafe_b64encode(key_bytes)

  

# --- 3. Execution ---

  

# Step A: Convert hex back to bytes

encrypted_data = bytes.fromhex(hex_string)

print(f"Encrypted Bytes: {encrypted_data[:15]}...")

  

# Step B: Calculate the exact TOTP code for that time

# Note: integer division // 30 is the standard TOTP interval

time_counter = int(target_timestamp // 30)

totp_code = get_hotp_token(secret_base32, time_counter)

print(f"Recovered Seed (TOTP): {totp_code}")

  

# Step C: Generate the Key

key = get_encryption_key(totp_code)

print(f"Recovered Key: {key.decode()}")

  

# Step D: Decrypt

try:

f = Fernet(key)

decrypted_message = f.decrypt(encrypted_data)

print("\nSUCCESS! Decrypted Message:")

print("-" * 30)

print(decrypted_message.decode())

print("-" * 30)

except Exception as e:

print(f"\nDecryption Failed: {e}")
```

with the output containing our flag:

![[Pasted image 20260202170656.png]]

### My solution

This was the solution, and it was nice to be done with this after two hours of looking at obfuscated lambda statement code, but for me there was an issue - I still didn't know how to solve the problem myself, and further neither did he, since he used generative AI to create the script above.

After a few days I came back to finally extract the flag, but now with more time, and a better understanding of how the code works - I haven't actually covered that yet, so lets do that:

### How it works

This code takes a timestamp, and some string to create an encryption key, which we can call TOPT, a Time-based one-time password.

We then use Fernet to encrypt our data with our key. Fernet is a symmetric encryption algorithm, which basically means that the same key can be used for both encryption and decryption. Because Fernet is symmetric, if we are able to find the key (which we are able to generate with the given code and a timestamp), then we can decode our data.

Using the original lambda code to ensure our decryption is accurate, lets do that:

First, lets refactor the code to allow for a specific timestamp & data to be used.
Second, lets add some code to decrypt any data we give it
Third, the encrypted data has been converted to hex, so lets undo that quickly:
67414141414142706542504d4e597439356876444265624e356d77727974597663724a5145666c70475f7261316372616d63392d59312d4735576442646b6c6d48315a4d7474794b316f77484a70315064486c6d4b59386a5f764c75366b6131336a57443139375741485249674964355a7347524a62343d

becomes

gAAAAABpeBPMNYt95hvDBebN5mwrytYvcrJQEflpG_ra1cramc9-Y1-G5WdBdklmH1ZMttyK1owHJp1PdHlmKY8j_vLu6ka13jWD197WAHRIgId5ZsGRJb4=

Our new code:

```
from cryptography.fernet import Fernet

from ast import literal_eval

import hashlib

import struct

import base64 as bs64

import random

import time

import hmac

  

timestamp = 1769477068.8361542

  

htp = lambda s,c,d=6:(lambda h:str((struct.unpack('>I',h[(h[-1]&15):(h[-1]&15)+4])[0]&2147483647)%10**d).zfill(d))(hmac.new(s,struct.pack('>Q',c),hashlib.sha1).digest())

tpt = lambda s,t=30,d=6:htp(bs64.b32decode(s.upper()+'='*((8-len(s)%8)%8)),int(timestamp//t),d)

ctpttk=lambda t:(lambda b:bs64.urlsafe_b64encode(b).decode().encode())(random.getrandbits(256).to_bytes(32,'big') if not random.seed(t) else None)

rfd=lambda p:open(p,'rb').read()

e=lambda p:(lambda k,o,f,d:print(f.encrypt(d).hex()))("J5JVK6ZQNZSV6VDJNVSV6UDBMRZV6ZTJNRWGK4T5",(o:=tpt("J5JVK6ZQNZSV6VDJNVSV6UDBMRZV6ZTJNRWGK4T5")),Fernet(ctpttk(o)),rfd(p))

print(time.time())

  
  

f = Fernet(ctpttk(tpt("J5JVK6ZQNZSV6VDJNVSV6UDBMRZV6ZTJNRWGK4T5")))

  

data = "gAAAAABpeBPMNYt95hvDBebN5mwrytYvcrJQEflpG_ra1cramc9-Y1-G5WdBdklmH1ZMttyK1owHJp1PdHlmKY8j_vLu6ka13jWD197WAHRIgId5ZsGRJb4="

  

print(f.decrypt(data))
```

This should now spit out the decrypted data:

![[Pasted image 20260202172214.png]]

# What did we learn?

Normally I don't throw in these bits, but I think there is a lesson to be learned here - if all else fails, start with your assumptions:
- were they using the same os as me?
- was this made in the same language?
- were we taught this wrong?

It is likely that if we had checked our assumptions along the way, we would've spent far less time on this, but we got there in the end.