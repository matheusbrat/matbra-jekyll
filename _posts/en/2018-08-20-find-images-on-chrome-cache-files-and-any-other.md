---
layout: post
title: Find images on chrome cache files (or any other file!)
lang: en
description: Recover images from cache files on chrome or any other binary file
tags: image chrome jpg jpeg png python carving hacker file find
comments: true
--- 

Good night,

Recently I have deleted a few images from my image which the old link was broken on the last few days. I decided to try to find them on the Google Chrome Cache. 
The url `chrome://cache` was recently removed, but you can find your chrome cache files at: `/home/matheus/.cache/google-chrome/Default/Cache/`. 

If you open it as binary, you will see it is not a file directly. There is more information embeded in the file such as URL, headers, http status code and others. We could take a look on chrome source code to extract everything from the file, not only images. But to be honest I was lazy to dig into that because I had a very specific need in this case. [Chrome cache storage](https://github.com/chromium/chromium/tree/f18e79d901f56154f80eea1e2218544285e62623/content/browser/cache_storage)

Why not scan the cache files for the JPEG binary? We would need to know how to find the start/end of image. We will have:
- bytes 0xFF, 0xD8 indicate start of image
- bytes 0xFF, 0xD9 indicate end of image

OK. So how would we do this in python? 

Open the file as binary and check if there is a JFIF or EXIF marker on it. (Just trying to ignore files we can't process)
```
f = open(filepath, 'rb')

data = f.read()
if 'JFIF' not in data and 'Exif' not in data:
	return
```

Now let's iterate over all the bytes trying to find that specific sequence. To achieve this let's have a prev which will have the value of the previous byte, pos to know which position we're at and an array for SOI (Start of image) and EOI (End of Image) which will hold the positions for this markers. If the previous char is FF and the current one is D8, it will append to SOI, if it is D9 it will append to EOI.
```
prev = None
soi = []
eoi = []
pos = 0
for b in data:
	if prev is None:
		prev = b
		pos = pos + 1
		continue
	if prev == chr(0xFF):
		if b == chr(0xD8):
			soi.append(pos-1)
		elif b == chr(0xD9):
			eoi.append(pos-1)
	prev = b
	pos = pos + 1
```

We can get the SOI e EOI and save it. The only magic we will be doing here is getting the first SOI and the last SOI or EOI depending on each one is bigger.
```
path, filename = os.path.split(filepath)
file = open('{}/{}-{}.jpg'.format(OUTPUT_FOLDER, filename, 0), 'wb')
m1 = soi[0]
m2 = soi[-1] if soi[-1] > eoi[-1] else eoi[-1]
file.write(data[m1:m2])

file.close()

print(filename, "SOI", soi, len(soi))
print(filename, "EOI", eoi, len(eoi))
```
This code will save only one image. If you want you could iterate over the SOI and EOI and save multiple files. 


Would this be some kind of file carving? 

I hope this helps you!
Matheus


Get this script create the OUTPUT_FOLDER and run it as `python yourfile.py filetocheck`, this version should be able to handle multiple images inside the same file. Now you can check and output stream for instance.

```
import os
import glob
import sys

OUTPUT_FOLDER = "output-this2"


def save_file(data, path, filename, count, eoi, soi):
	file = open('{}/{}-{}.jpg'.format(OUTPUT_FOLDER, filename, count), 'wb')
	m1 = soi[0]
	m2 = soi[-1] if soi[-1] > eoi[-1] else eoi[-1]
	file.write(data[m1:m2])
	file.close()

def extract(filepath):
	count = 0
	f = open(filepath, 'rb')

	data = f.read()
	if 'JFIF' not in data and 'Exif' not in data:
		return

	path, filename = os.path.split(filepath)

	old_soi = []
	old_eoi = []
	prev = None
	soi = []
	eoi = []
	eoi_found = False
	pos = 0
	for b in data:
		if prev is None:
			prev = b
			pos = pos + 1
			continue
		if prev == chr(0xFF):
			if b == chr(0xD8):
				if eoi_found:
					save_file(data, path, filename, count, eoi, soi)
					old_soi = old_soi + soi
					old_eoi = old_eoi + eoi
					soi = []
					eoi = []
					count = count + 1
					eoi_found = False
				soi.append(pos-1)
			elif b == chr(0xD9):
				eoi.append(pos-1)
				eoi_found = True
		prev = b
		pos = pos + 1

	save_file(data, path, filename, count, eoi, soi)
	print(filename, "SOI", soi, len(old_soi))
	print(filename, "EOI", eoi, len(old_eoi))

def main():
	if len(sys.argv) < 2:
		sys.exit(1)

	extract(sys.argv[1])

if __name__=="__main__":
	main()
```

Reference:
[https://stackoverflow.com/questions/4585527/detect-eof-for-jpg-images](https://stackoverflow.com/questions/4585527/detect-eof-for-jpg-images)
