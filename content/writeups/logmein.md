---
title: "Logmein"
date: 2024-01-21T22:26:56+01:00
tags: ["CTF","Writeups","HACKDAY 2024 Qualification","Forensic"]
author: "Sirius14"
showToc: true
TocOpen: false
draft: false
hidemeta: false
comments: false
description: "HACKDAY 2024 Qualification Crypto challenge by ZEN"
disableHLJS: false # to disable highlightjs
disableShare: false
hideSummary: false
searchHidden: true
ShowReadingTime: true
ShowBreadCrumbs: false
ShowPostNavLinks: false
ShowWordCount: false
ShowRssButtonInSectionTermList: false
UseHugoToc: true
pygmentsUseClasses: true
pygmentsCodeFences: true
cover:
    image: "/Sirius14_Blog/img/logo.png" # image path/url
    alt: "<alt text>" # alt text
    caption: "<text>" # display caption under cover
    relative: false # when using page bundles set this to true
    hidden: true # only hide on current single page
---

Log in by using the identifiers found in the "Baby Reverse Engineering" test. The person with these identifiers has been identified as working for HACK107, and we also know from trusted sources by our double agent that these computer defense capabilities are highly exploitable. You should therefore be able to take advantage of it to find interesting information and retrieve the flag to get your hand on his data.

`ssh MemeMaster@challenges.hackday.fr -p 50391`

## First look

To start this challenge we need to connect to the SSH server.
Once logged in we can visit home of different users :
- `BootMaster`
- `BruteNinja`
- `MemeMaster` (our home)

In BootMaster home I found a interesting file named : cipheredflag

Here is content of cipheredflag:

{{<highlight txt>}}
HQUXGAIEChoVEAdkBBIfFgIPEQQIHBQID2QHChoKBR8Je1NCFG1rFAocGRUWAABhEhJhFwUXYQ0ZBwQYEEsYBRQHDGQFCgdhCAYRBQ1oFhAfCgoVBW1jHmQOCAVhEhMUEnkcCmMYHhsMDRQQDxARGmgRCA4VFQpoFQ0YZA8SHxJjAB5hDQYSYwUXEBVoChlrCRIcCQcWHwdteQcPFg4KYRYZDGMbDwQIHBIPaxUVeXxhGgoVFHkEEg5rDwcDi8kLDRcMeR0FYxkVBwwFb2NBTksMHW9jBg8EDBkRYxgABgwMEhANEQUeQgAOHgAGeRkFYx4ABRIAEBceDBJ5HxIXBQ8NeQMbYwYPBAwZEWMYAAYMDBIQDREFHmRhEg8IYR0DFRYOEQUeaBUNGhBhFxthDRoIYRwZDBdmEQcSDGEMGhASH2gHEgcREA0DBw5nZBR5GwUXGAISFWgPFgZkDxIfEmMfD2EKHxMXCGQVFxthBRoIFAgcCgUGamEMaBUIAA1hDQYSYwAWExIOFQIPERUaaBUIaxYRCA0SYxwRBB8DEw0AABIUaBQHawwQHx0KAgZkEwgOFQ5rDxt5HA8QawoKGAUSbWsRYQgYDAhrEQUUHBATGxcNeRlhBRgTCg0bEQBrAQodHAcIGxQQARgSYwoJGXkdFhEbFQUNaBQHaxAPCmgZEg4NYRISYRcFF2EDGRYWGwdhHAMWEQoQEh9mS0keERR3aBEIDhUReRkNFB4ASw0GFA5rAQoOGgAXGAJhFhkMYxwPBwpoBwgTBQwNaBUNGgBhCh4TEA4QEhVmb21rEA8KaBYSDgoUHRthDBoJYRgaYRcAZA0IHBJjGgANeRwPEGsWBwwAFBMYAxIUaA4QDRdhFhsRE2sJEh4fEgcfFw13aA8IBBcJCg5tYxpkDQobE2MYAAAOGwcSHxEKHWgKGWsQDwpoGRIODQALaAwABhASDmgQExsPDgoEYRQYZBUSaBsWDwhhCGgVFR9kGwwYEmMODwUNGRQHHgAGeRwPEGsVDQ4dBRYGEAcIHAoFbAlhHA4SDxgAFQwZEQ5rEQV5HA8Qaw8RFRsMF2sJFQwYEWMDFREMBGEbGgEIGBpvYxpkDgobCGMbFRUKDmESDwhhDQYSYxEREQpoDggKFA15BhALGGQZChsFYwcXEQocEg9rFQANAxYSHxEECBgRAGVOSwwAb2MAHAMKDRUWAxdhHxsQBgUXDVMHEmMNFwQSABIFGAhhDQYSYw0XDBIJBwYYCWEWG2EHGBcNCgRhFwBkFggBEmMABQd5HwoLGGphDQYSDhhkBwoFCgINARIUaBAFGGQKHWgKAg1kABQJEBNrAwoSDxEQawgHDAASbWsTCh8bYRYPCRUfCQQXHg8FFGgVCGseCgkYCgxlTksbAw8HaxxvU0IPEg4NDQgLOgYFdwQQFwJzCgIeCXgGDjZO
{{</highlight>}}

It seems to be encoded in base64, but once decoded it was unreadable.

At this moment I had no idea what I was supposed to do so I downloaded linpeas and ran it, but nothing special

## Log

The next day I read again the title : LOGMEIN

The log I don't check the /var/log !!!
I saw this file : `docker-build.log` which was very interesting
![docker-build.log](/Sirius14_Blog/img/writeups/logmein_1.png)

Nevertheless, as you can see I don't have all the content of this file. So I decided to exfiltrate it with base64 command: `base64 docker-build.log`.
Like this I have all the content. I decoded and I found this part which help us to understand how cipheredflag file was created:

{{<highlight txt>}}
# Some content ...
Step 11/17 : RUN "python3" "affinecipher.py" "clearflag.txt"
 ---> Running in ae3f947636db
Removing intermediate container ae3f947636db
 ---> fed9b9a84d51
Step 12/17 : RUN "python3" "xorwithkey.py" "clearflag.txt" "HACKDAY"
 ---> Running in f01dd4c4e5e3
Removing intermediate container f01dd4c4e5e3
 ---> 638d5a463650
Step 13/17 : RUN "python3" "base64this.py" "clearflag.txt"
 ---> Running in 50c0691102ad
Removing intermediate container 50c0691102ad
 ---> 11f892281b42
Step 14/17 : RUN mv clearflag.txt cipheredflag
 ---> Running in c0aa56107988
Removing intermediate container c0aa56107988
 ---> d8ccf043b854
Step 15/17 : RUN rm *.py
# Other content ...
{{</highlight>}}

Thanks to this log we can determine all the future steps to do:
- decode cipheredflag from is base64 encoding
- xor the result with the key `HACKDAY`
- bruteforce affine cipher parameters

## Decode, decrypt and Flag

With this wonderful tool called [cyberchef](https://gchq.github.io/CyberChef/#recipe=From_Base64('A-Za-z0-9%2B/%3D',true,false)XOR(%7B'option':'UTF8','string':'HACKDAY'%7D,'Standard',false)&input=SFFVWEdBSUVDaG9WRUFka0JCSWZGZ0lQRVFRSUhCUUlEMlFIQ2hvS0JSOEplMU5DRkcxckZBb2NHUlVXQUFCaEVoSmhGd1VYWVEwWkJ3UVlFRXNZQlJRSERHUUZDZ2RoQ0FZUkJRMW9GaEFmQ2dvVkJXMWpIbVFPQ0FWaEVoTVVFbmtjQ21NWUhoc01EUlFRRHhBUkdtZ1JDQTRWRlFwb0ZRMFlaQThTSHhKakFCNWhEUVlTWXdVWEVCVm9DaGxyQ1JJY0NRY1dId2R0ZVFjUEZnNEtZUllaREdNYkR3UUlIQklQYXhVVmVYeGhHZ29WRkhrRUVnNXJEd2NEaThrTERSY01lUjBGWXhrVkJ3d0ZiMk5CVGtzTUhXOWpCZzhFREJrUll4Z0FCZ3dNRWhBTkVRVWVRZ0FPSGdBR2VSa0ZZeDRBQlJJQUVCY2VEQko1SHhJWEJROE5lUU1iWXdZUEJBd1pFV01ZQUFZTURCSVFEUkVGSG1SaEVnOElZUjBERlJZT0VRVWVhQlVOR2hCaEZ4dGhEUm9JWVJ3WkRCZG1FUWNTREdFTUdoQVNIMmdIRWdjUkVBMERCdzVuWkJSNUd3VVhHQUlTRldnUEZnWmtEeElmRW1NZkQyRUtIeE1YQ0dRVkZ4dGhCUm9JRkFnY0NnVUdhbUVNYUJVSUFBMWhEUVlTWXdBV0V4SU9GUUlQRVJVYWFCVUlheFlSQ0EwU1l4d1JCQjhERXcwQUFCSVVhQlFIYXd3UUh4MEtBZ1prRXdnT0ZRNXJEeHQ1SEE4UWF3b0tHQVVTYldzUllRZ1lEQWhyRVFVVUhCQVRHeGNOZVJsaEJSZ1RDZzBiRVFCckFRb2RIQWNJR3hRUUFSZ1NZd29KR1hrZEZoRWJGUVVOYUJRSGF4QVBDbWdaRWc0TllSSVNZUmNGRjJFREdSWVdHd2RoSEFNV0VRb1FFaDltUzBrZUVSUjNhQkVJRGhVUmVSa05GQjRBU3cwR0ZBNXJBUW9PR2dBWEdBSmhGaGtNWXh3UEJ3cG9Cd2dUQlF3TmFCVU5HZ0JoQ2g0VEVBNFFFaFZtYjIxckVBOEthQllTRGdvVUhSdGhEQm9KWVJnYVlSY0FaQTBJSEJKakdnQU5lUndQRUdzV0J3d0FGQk1ZQXhJVWFBNFFEUmRoRmhzUkUyc0pFaDRmRWdjZkZ3MTNhQThJQkJjSkNnNXRZeHBrRFFvYkUyTVlBQUFPR3djU0h4RUtIV2dLR1dzUUR3cG9HUklPRFFBTGFBd0FCaEFTRG1nUUV4c1BEZ29FWVJRWVpCVVNhQnNXRHdoaENHZ1ZGUjlrR3d3WUVtTU9Ed1VOR1JRSEhnQUdlUndQRUdzVkRRNGRCUllHRUFjSUhBb0ZiQWxoSEE0U0R4Z0FGUXdaRVE1ckVRVjVIQThRYXc4UkZSc01GMnNKRlF3WUVXTURGUkVNQkdFYkdnRUlHQnB2WXhwa0Rnb2JDR01iRlJVS0RtRVNEd2hoRFFZU1l4RVJFUXBvRGdnS0ZBMTVCaEFMR0dRWkNoc0ZZd2NYRVFvY0VnOXJGUUFOQXhZU0h4RUVDQmdSQUdWT1N3d0FiMk1BSEFNS0RSVVdBeGRoSHhzUUJnVVhEVk1IRW1NTkZ3UVNBQklGR0FoaERRWVNZdzBYREJJSkJ3WVlDV0VXRzJFSEdCY05DZ1JoRndCa0ZnZ0JFbU1BQlFkNUh3b0xHR3BoRFFZU0RoaGtCd29GQ2dJTkFSSVVhQkFGR0dRS0hXZ0tBZzFrQUJRSkVCTnJBd29TRHhFUWF3Z0hEQUFTYldzVENoOGJZUllQQ1JVZkNRUVhIZzhGRkdnVkNHc2VDZ2tZQ2d4bFRrc2JBdzhIYXh4dlUwSVBFZzRORFFnTE9nWUZkd1FRRndKekNnSWVDWGdHRGpaTw), we can do the first 2 task. Then using [dcode](https://www.dcode.fr/chiffre-affine) to bruteforce affine parameters (a=7, b=13).

Then we can Flag with : __HACKDAY{CH3CK_Y0UR_L0GS}__