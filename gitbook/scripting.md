# Scripting

TryHackMe - [**Scripting**](https://tryhackme.com/room/scripting) Write-Up

This room python blah blah balh, purposes blah, overview

1. Easy
2. Medium
3. Hard

## Easy

background on plan blah blah

```text
#!/usr/bin/env python

from base64 import *

b64 = open("/home/slickmmarek/Documents/OSCP/python/b64.txt", "r")
b64r = b64.read()

for i in range(0,50):
	b64r = b64decode(b64r)

b64.close()
print(b64r)
```

post description and details

post pic of result here

## Medium

## Hard

