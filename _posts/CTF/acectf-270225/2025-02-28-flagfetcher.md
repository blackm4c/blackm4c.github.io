---
title: ACECTF 2025 - web - Flag Fetcher
date: 2025-02-28 20:00:00 +0530
categories: [CTF, acectf-2025]
tags: [web, analysis, console]     # TAG names should always be lowercase
description: Extracted the flag by analyzing console errors showing failed fetch requests
toc: true
# comments: false
# pin: true
# published: false
---

---

## Challenge Description

```
Hey guys, I created a flag fetcher using some web stacks & technologies. It was supposed to fetch the flag.webp image file which contains the flag, but something went wrong. Can you check it? Maybe just get the flag—I don’t really care if you fix it or not.
```

## Analysis

When I visited the URL, it initially loaded with the path `/Flag-Fetcher/` and then redirected to `/flag.webp`.

### Initial Investigation

1. **Checked Cookies & Local Storage** – Nothing useful.
2. **Inspected Page Source & Website Elements** – No hidden clues.
3. **Tried Path Traversal** – Attempted to fetch files directly, but it didn’t work.
4. **Checked Apache Version** – Found it was the latest version and not vulnerable to known exploits.
5. **Downloaded `flag.webp`** – The file contained no useful information.

### Finding the Clue

While inspecting the browser console, I noticed multiple errors related to file fetching. The site was attempting to retrieve specific directory paths but returned **404 Not Found** errors.

### Extracting the Flag

By carefully checking the errors, I realized the system was fetching **each letter of the flag separately**. By reconstructing the flag based on these failed requests, I was able to piece together the complete flag.

![Desktop View](/assets/img/ctf/acectf2025/flagfetcher.png){: .w-75 .shadow .rounded-20 w='1212' h='668'}

### Lessons Learned

- **Always check the browser console for errors** – Debug messages might reveal hidden clues.
- **404 errors can still provide useful information** – Pay attention to missing files.
- **Web applications may unintentionally leak data** – Understanding how files are loaded can help retrieve hidden information.



