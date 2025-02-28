---
title: ACECTF 2025 - web - Bucket List
date: 2025-02-28 20:00:00 +0530
categories: [CTF, acectf-2025]
tags: [web, aws, amazon-s3]     # TAG names should always be lowercase
description: misconfigured S3 buckets can expose flag
toc: true
# comments: false
# pin: true
# published: false
---

---


## Initial Analysis

### Challenge Description

```
You know what a bucket list is? It's a list of wishes people want to achieve before they leave this world. But isn't it ironic? How can you know when your time will come? Instead of waiting, it's better to enjoy every moment and seize every opportunity.

One of my wishes, though, is to pet a cat. Do you mind checking this one out? So cute.
```

### Investigating the URL

The given URL:
```
https://opening-account-acectf.s3.ap-south-1.amazonaws.com/fun/can_we_get_some_dogs/026.jpeg
```
This clearly indicates that the challenge is hosted on **Amazon S3**.

The URL opened an image of a cute cat. To explore further, I attempted to navigate to:
```
https://opening-account-acectf.s3.ap-south-1.amazonaws.com/fun/can_we_get_some_dogs
```
But this returned an error:
```xml
<Error>
<Code>NoSuchKey</Code>
<Message>The specified key does not exist.</Message>
<Key>fun/can_we_get_some_dogs/</Key>
</Error>
```

### Exploring the S3 Bucket

Next, I navigated to `/fun`, which triggered a file download. However, the file was empty (0 KB), providing no useful clues.

Then, I accessed the root directory and **discovered a full list of the bucket’s contents**. Among the directories, I found some interesting paths:

- `l33t-h4x0r/flag_here/041.png`
- `l33t-h4x0r/flag_here/078.jpeg`
- `l33t-h4x0r/flag_here/082.jpeg`

These were just random images with no useful information. However, I kept looking and found something promising:

### Finding the Secret File

One interesting directory stood out:
```
cry-for-me/acectf/secret.txt
```
When I accessed this file, it contained encoded data. It looked like **Base64** encoding. Decoding it revealed the flag:

```
QUNFQ1RGezdoM180dzVfMTVfbTE1YzBuZjE2dXIzZH0=
Decoded: ACECTF{7h3_4w5_15_m15c0nf16ur3d}
```

## Conclusion

This challenge demonstrated how **misconfigured S3 buckets** can expose sensitive data. By listing and accessing unrestricted files, we were able to retrieve the flag.






