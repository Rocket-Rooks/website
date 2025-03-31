---
date: 2025-03-31T15:31:15+11:00
modified: 2025-03-31T15:31:15+11:00
author: snth
title: On Thin Ice Writeup - SwampCTF 2025
tags: writeup ctf SwampCTF OSINT
series: SwampCTF2025
---

| CTF:            | SwampCTF    |
| --------------- | ----------- |
| Challenge Type: | OSINT       |
| Challenge Name: | On Thin Ice |
| Author:         | snth        |

![](/img/swampctf2025_oti_challenge.png)

![](/img/swampctf2025_oti_blank.jpg)

The image is very dark and nothing visually identifiable. 

First thing to check is whether there is anything in the EXIF data that may provide information or clues.

Using exiftool to extract the exif data:
```
ExifTool Version Number         : 13.10
File Name                       : blank.jpg
Directory                       : .
File Size                       : 3.1 kB
File Modification Date/Time     : 2025:03:29 08:13:54+11:00
File Access Date/Time           : 2025:03:29 08:14:21+11:00
File Inode Change Date/Time     : 2025:03:29 08:14:15+11:00
File Permissions                : -rwxrwx---
File Type                       : JPEG
File Type Extension             : jpg
MIME Type                       : image/jpeg
Exif Byte Order                 : Big-endian (Motorola, MM)
About                           : uuid:faf5bdd5-ba3d-11da-ad31-d33d75182f1b
Title                           : D3 A6 D0 BA D0 BC D1 8B D1 81 D3 A7 D0 B4 20 D0 B2 D0 BE D1 81 D1 8C D0 BA D0 BE D0 B2 2E 20 D0 9C D0 B5 D0 B7 D0 B4 D0 BB D1 83 D0 BD 2E
Description                     : D3 A6 D0 BA D0 BC D1 8B D1 81 D3 A7 D0 B4 20 D0 B2 D0 BE D1 81 D1 8C D0 BA D0 BE D0 B2 2E 20 D0 9C D0 B5 D0 B7 D0 B4 D0 BB D1 83 D0 BD 2E
Image Width                     : 356
Image Height                    : 200
Encoding Process                : Progressive DCT, Huffman coding
Bits Per Sample                 : 8
Colour Components                : 1
Image Size                      : 356x200
Megapixels                      : 0.071
```
The Title and Description contains identical hex values.

Using CyberChef to decode the hex into text:
![](/img/swampctf2025_oti_cyberchef.png)

Decoded text:
Ӧкмысӧд воськов. Мездлун.

Translated to English:
Step eight. Freedom.

Searched "Step eight. Freedom."
It's a Call of Duty reference : https://www.giantbomb.com/vorkuta/3035-3439/

Searched "The Vorkuta Gulag" - used the Coordinates referenced in the Wikipedia page to search Google Maps. Searching "Vorkutlag" in Google Maps will achieve the same result.
https://en.wikipedia.org/wiki/Vorkutlag

Coordinates: 67° 29' 48.4080 _N 64° 3' 38.2968_ E

Using the 'nearby search' feature, used the search term 'skating' 

Found УСЗК "ОЛИМП" which is rated pretty decent - as per challenge description. 

Sorted by 'most recent reviews': 
![](/img/swampctf2025_oti_reviews.png)

Flag found in the name of the most recent review.

Flag:
`swampCTF{ForUM4sOnN0tForM3}`