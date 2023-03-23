+++
author = "Lonus"
title = "Extracting the binary data from a photo"
date = "2023-03-23"
description = "Write up of how I solved the Glory of the Garden picoCTF challenge"
tags = [
    "write-up",
    "ctfs",
    "linux",
    "capture-the-flag",
    "hacking",
]
categories = [
    "Hacking",
    "CTFs",
    "picoCTF",
    "Challenges",
    "Linux",
]
series = ["picoCTF-challenges"]
aliases = ["extract-data"]
image = "coding-hero.jpg"
+++

> This is just a write up on resolving picoCTF(2021) Glory of the Garden challenge.

First I tried running **exiftool** which showed me the metadata of the photo where I noticed that the lines with the following:

```shell
Red Tone Reproduction Curve     : (Binary data 2060 bytes, use -b option to extract)
Green Tone Reproduction Curve   : (Binary data 2060 bytes, use -b option to extract)
Blue Tone Reproduction Curve    : (Binary data 2060 bytes, use -b option to extract)
```

Meaning the photo has son binary data attached to it, now the question is

## How do I extract that data?

After some digging on Youtube School I found various ways to do it

> The challenge hint it is about using a hex editor which it will help you extract the data. Using that method you will have to scroll and found the flag.

## Using **strings** command

The first way and the most easy one is using **strings** command. Using this command it will show you only the ASCII representation of the strings inside the binary data of the image.

> strings - print the sequences of printable characters in files

Command:

```shell
strings garden.jpg
```

Output:

```shell
....
%zR}
Tw1<Oi
yg9S
F7v]
m o5Xex#
V8L;
<]zV&
ZqKvu
7g4js'
wae:uc(>YwG
6       `A
xhS~
wM=GV
gDau%~
,~J|
u)(])F
={~5
h--@3
cZi-
M(.I
]hWP&
jc#k
=7g&
mjx/
s\]|."Ue
\qZf
--------------
Here is a flag "picoCTF{more_than_m33ts_the_3y3eBdBd2cc}"
--------------
```

## Using **xxd** command

The second way to do it will be using the **xxd** command. Using it will start dumping in to the shell all the hex data of the image including the flag.

> xxd creates a hex dump of a given file or standard input. It can also convert a hex dump back to

    its original binary form.  Like uuencode(1) and uudecode(1) it allows the transmission  of  binary
    data  in a `mail-safe' ASCII representation, but has the advantage of decoding to standard output. Moreover, it can be used to perform binary file patching.

Command:

```shell
xxd garden.jpg
```

Output:

```shell
002304a0: a9e2 cfba bf51 5c75 a71b b6e3 7b1f 97cb  .....Q\u....{...
002304b0: 25a5 f5c5 4bda c9a9 6ecf 7ed4 fc4f e1cd  %...K...n.~..O..
002304c0: 2919 e6bb 591b 9c2a 73d2 be71 bdf8 977d  )...Y..*s..q...}
002304d0: e27d 6574 fd3d 1a08 233f bd6e 99c7 6af9  .}et.=..#?.n..j.
002304e0: e2f7 fd6f fc09 ff00 9d6f fc2b ff00 9096  ...o.....o.+....
002304f0: adff 005d ff00 a0ae 7a51 8d4f 68f5 568a  ...]....zQ.Oh.V.
00230500: d9f9 9f63 4b2b c1e5 daf2 7b59 db49 4ba3  ...cK+....{Y.IK.
00230510: f43e b881 5e30 1060 8030 47d6 bacb 58cb  .>..^0.`.0G...X.
00230520: 1046 07b5 7216 df7e 5ff7 c576 363f ebab  .F..r..~_..v6?..
00230530: b70d 18ce 3ccd 6a7e 6b8d af56 b579 39ca  ....<.j~k..V.y9.
00230540: eeef 53ae 8620 31b8 751f 9514 f7fb cff5  ..S.. 1.u.......
00230550: a2bb bdac 9687 98e4 d3b2 e87f ffd9 4865  ..............He
00230560: 7265 2069 7320 6120 666c 6167 2022 7069  re is a flag "pi
00230570: 636f 4354 467b 6d6f 7265 5f74 6861 6e5f  coCTF{more_than_
00230580: 6d33 3374 735f 7468 655f 3379 3365 4264  m33ts_the_3y3eBd
00230590: 4264 3263 637d 220a                      Bd2cc}".
```

And that's it !! got the flag but not stopping I kept looking around the internet and found some extra steps to make the output of the **strings** fancier, Like only printing the flag passing**tail** command to the pipeline or getting only the flag without the extra text passing the **cut** command or making a simple bash script to automate everything.

## Extra steps:

### Passing the **tail** command to the pipeline

`What does the tail command do?`

> tail - Print the last 10 lines of each FILE to standard output. With more than one FILE, precede each with a header giving the file name.

       -n, --lines=[+]NUM
              output the last NUM lines, instead of the last 10; or use -n +NUM to output  starting  with
              line NUM

Since the flag is on the last line of the data I passed the following to the pipeline

```shell
tail -n 1
```

Now the **strings** command should look like this:

```shell
strings garden.jpg | tail -n 1
```

and the Output:

```shell
Here is a flag "picoCTF{more_than_m33ts_the_3y3eBdBd2cc}"
```

### Passing the **cut** command to the pipeline

`What does the cut command do?`

> cut - remove sections from each line of files
> Print selected parts of lines from each FILE to standard output. - d, --delimiter=DELIM

    			use DELIM instead of TAB for field delimiter
    	- f, --fields=LIST
    			 select only these fields; also print any line that contains no delimiter character, unless the -s option is specified

So as you can see the entire string has **"Here is a flag"** and the flag is surrounded by **""** so using the **-d** and **-f** come in handy to tell the command what part of the string I want so by passing **-d** flag like this:

```shell
-d '"'
```

this way the **cut** command it'll be set **(")** as the delimiter to split the string into fields, the double quotes is used to separate and distinguish one section of text from another within a larger body of text and the way **cut -d** works the **-f** flags needs to be specified or else will throw this error:

```shel
cut: you must specify a list of bytes, characters, or fields
```

So the way to do this is finding on what field is the flag **cut** works kind of like this:

```shell
Here is the Flag  "picoCTF{more_than_m33ts_the_3y3eBdBd2cc}"
|______1________| |___________________2____________________|
```

When **cut** processes the input string, it splits it into fields based on the delimiter. In the case of the string **Here is the Flag "picoCTF{more_than_m33ts_the_3y3eBdBd2cc}"**, the delimiter (") occurs twice in the string, so cut splits it into three fields:

```
1. `Here is the flag`
2. "picoCTF{more_than_m33ts_the_3y3eBdBd2cc}"
3. empty field
```

After **cut** has split the input string into fields, you can use the **-f** option to specify which fields you want to keep. In this case, we want to keep the second field, which contains the flag. So we use the **-f 2** option to tell **cut** to keep only the second field.

So, the **-d '"'** option is not removing the double quotes from the fields. Rather, it is specifying the character that should be used as the delimiter to separate the input string into fields. The full command should look like this now:

Command:

```bash
strings garden.jpg | tail -n 1 | cut -d '"' -f2
```

Output:

```bash
picoCTF{more_than_m33ts_the_3y3eBdBd2cc}
```

And to automate all this I just pasted the command into a bash file so instead of writing all that line all I have to do is run the file and pass the name of the photo.
So as for the file just create an **.sh** file and paste the code but instead of specifying the photo name I change for a positional parameter so it can be run this way:

```bash
./<filename>.sh garden.jpg
```

So the modified command looks like this:

```bash
#!/bin/bash
strings "$1" | tail -n 1 | cut -d '"' -f2
```

In this updated command, **$1** is a positional parameter that refers to the first argument passed to the script when it is executed. So, if you execute the script with the command **./bashfile.sh garden.jpg,** then **$1** will be replaced with **garden.jpg** in the script. This allows you to run the script with different image file names without having to edit the script itself.
