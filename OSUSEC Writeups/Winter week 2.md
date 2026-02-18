porygons & polyglots

# Description
Flag will need to be wrapped in osu{}

# Flag 1

The challenge this week was a wav audio file.
yep, thats it
definitely nothing here

anyways if you look at the file under a hex editor you see this initial set of characters that define a wav audio file:

![[Pasted image 20260113102626.png]]

the idea of this challenge is polyglots, where one file actually runs as two (pdf that runs a program, text file with an image embedded, etc.) Because of this I looked at the end of the file for another filetype's signature:

![[Pasted image 20260113102908.png]]

Here we see a set of "super_secret_hidden_files" and a big PK, if we find which filetype corresponds to PK we get a zip file, which when extracted gets us this:

![[Pasted image 20260113103158.png]]

These three files are really weird. for one, porygon z has three detected file types in it:

![[Pasted image 20260113103454.png]]

second, the two image files (porygon-z and porygon) are of porygon, a pokemon known for being kinda weird and having three forms, this reminds me that we're working with polyglots and I start to think "wow, all these files are polyglots!", in fact they are.a

lets take a quick look at the other two files before we get started with the scavenger hunt before us.

porygon also has three filetypes detected - but when I checked during the challenge I didn't see this:

![[Pasted image 20260113104140.png]]

porygon2 has a LOT of stuff going on, more of that zlib data:

![[Pasted image 20260113104219.png]]

Now that we have some starting points, lets go through the files one by one to find our flag pieces

### Porygon

As we recall, porygon was detected to have a png, zlib, and mp3 file in it - here's the image:

![[Pasted image 20260113104505.png]]

The hex data has some clues:

![[Pasted image 20260113104556.png]]

It seems to have a comment, with the contents "Flag Fraction 2: lviz", if we use exiftool we see the actual comment:

![[Pasted image 20260113104714.png]]

Flag fraction 2: lvi

Looking through the rest of the file we find no other clues, so its time to view it as an mp3 file. When we do this we get some morse code -....---, if we send it to a morse code analyzing website it spits out "EVO", this is almost certainly a flag fraction, but since no number is attached we'll keep it in mind and move on.

### Porygon 2

This file was just detected to have a pdf file & a zlib compressed file - but over 10 of them.

Let's start with the pdf file - Opening it up shows us a giant list of all the pokemon, this will probably be useful

![[Pasted image 20260126164206.png]]

Looking in the raw hex data, we see a few eye-catching details:

![[Pasted image 20260126164350.png]]

First is the specific pokemon, then later on is a very clearly encoded string, let's start with how to get a flag fragment from the pokemon.

If you find each pokemon's respective number in the pdf file, you get 70 105 108. Converting these numbers to their ascii characters gets Fil - Probably another flag fragment, but we'll figure out where it goes later.

Next is that large string of text - "ZmxhZyBmcmFjdGlvbiAzOiBuZ18=", the large variety of values in the string tells me its probably base64 encoded, so let's unravel that with cyberchef:

![[Pasted image 20260126164817.png]]

Pretty clearly the flag fraction we were looking for, now telling us that our flag looks something like: lving_ , and with two random flag fractions mixed in somewhere, EVO & Fil.

### Porygon-z

First let's recall that a png, pdf, and jpeg were all detected in the file. The first thing I did was open it, showing us a beautiful rendition of porygon-z

![[Pasted image 20260126165334.png]]

Now onto the hex data, where it was instant to detect some strange text, in the clear format of a flag fraction:

![[Pasted image 20260126165417.png]]

It says tEXtComment here so lets see if exiftool can spit out the exact string we need to decode:

![[Pasted image 20260126165547.png]]

Now we have our encoded string, note how the spaces, & colon were retained - This gives me a feeling that this might be encoded with a cypher, not a full encoding algorithm. This is further clear because of how well this lines up with how a caesar cypher works - for the uninitiated, a caesar cypher works by shifting each letter by a set amount. Some dead giveaways are that the length of the words is retained, and that the letter count & uniqueness is retained - that is to say that f's stay as f's, but switched for another letter. We can shift the letters until we find something familiar, in this case "flag fraction", if the first part lines up, the second part will too.

![[Pasted image 20260126170212.png]]

At 13 characters shifted, we get our 6th flag fragment: pes

This gives us some more info, and our flag is a little more formed now
lving_pes
\+ EVO & FIL

At this point we had been stuck looking through these files for two hours, and we were given these hints which really helped solve some of the fragments

![[Pasted image 20260126170444.png]]

Highlighting the entirety of each file lets us find our 5th flag fragment, ety, hidden in the pdf version of porygon-z

![[Pasted image 20260126170737.png]]

Its in white text, but copying fixes that.

### Putting it all together

With that we've found that our flag so far consists of:

lving_etypes

with leftover bits EVO & Fil. from here we can slot them in where they go to reveal our flag: 

osu{evolving_filetypes}