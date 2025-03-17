---
title: Three Words 1 Writeup
draft: false
modified: 2025-03-17T12:19:18+11:00
date: 2025-03-17T10:42:04+11:00
author: froakie10
tags:
  - UTCTF_2025
  - writeup
  - ctf
---
> This writeup is also available on [Medium.com](https://medium.com/@great.bj.shark/three-words-1-writeup-ea6167c72839)

In OSINT (Open Source Intelligence) Capture The Flag (CTF) challenges, even the smallest details — such as identifying the type of tree in an image — can provide crucial insights.

In one particular challenge, we were given an image featuring an oak tree next to a sign displaying the name “Hackerman,” with the letters “HB” above it. My initial search yielded no results, but a teammate successfully identified the location: the Norman Hackerman Building at the University of Texas at Austin.

The next objective was to determine the exact location of the sign. Using satellite imagery, I identified a silver statue near what appeared to be the building’s entrance. Upon further investigation via Google Street View, I confirmed that the statue was composed of multiple canoes. Nearby, I located the Norman Hackerman sign, verifying that we were on the right track.

At this stage, we needed to carefully analyze the original image for additional clues. Key details such as a “No Parking” sign and a distinctive forked light pole helped narrow down the precise location.

The final challenge involved pinpointing the exact coordinates of the flag using _What3words_, a platform that divides the world into 3-meter-by-3-meter squares. Given the vast number of possible locations, this step proved particularly challenging. Initially, we suspected the Norman Hackerman sign itself might mark the flag’s location, but this was not the case. After extensive teamwork and analysis, we successfully identified the correct position — the entrance door of the university.

This challenge reinforced the importance of meticulous observation, collaborative problem-solving, and leveraging multiple OSINT tools to achieve accurate geolocation.
