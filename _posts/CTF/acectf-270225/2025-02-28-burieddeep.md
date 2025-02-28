---
title: ACECTF 2025 - web - Buried Deep
date: 2025-02-28 20:00:00 +0530
categories: [CTF, acectf-2025]
tags: [web, enumeration, cryptography]     # TAG names should always be lowercase
description: A web exploitation challenge involving enumeration and cryptography.
toc: true
# comments: false
# pin: true
# published: false
---

---

## Initial Analysis

### Challenge Description

```
"I’m not a hacker. I’m just someone who wants to make the world a little better. But the world isn’t going to change itself."

Submit your answer in the following format: ACECTF{3x4mpl3_fl4g}

The flag content should be in lowercase letters only.
```

Upon visiting the challenge website, I noticed a long paragraph that contained some subtle hints.

![Desktop View](/assets/img/ctf/acectf2025/burieddeep.png){: .w-75 .shadow .rounded-20 w='1212' h='668'}

## Enumeration and Analysis

I began by checking cookies, local storage, and inspecting the website’s source code. Before attempting directory brute-forcing, I checked the `robots.txt` file. The contents were as follows:

```
# Hey there, you're not a robot, yet I see you sniffing through this file 😡
# Now get off my lawn! 🚫

Disallow: /secret/
Disallow: /hidden/
Disallow: /cryptic/
Disallow: /forbidden/
Disallow: /pvt/
Disallow: /buried/
Disallow: /underground/
Disallow: /secret_path/
Disallow: /hidden_flag/
Disallow: /buried_flag/
Disallow: /encrypted/
```

These directories seemed interesting, so I explored them one by one. Each directory contained hints that led me closer to the flag:

- **/secret** → "Nice try, but not quite! The flag’s just shy. Try again in the next path!"
- **/hidden** → "Flag’s not here, but you’re on the right track!"
- **/cryptic** → "Bingo? Not quite yet! Next path, let’s go!"
- **/forbidden** → "Almost there! Try a different route. The flag is playing hide and seek!"
- **/pvt** → "Flag not here! Keep calm and keep searching!"
- **/buried** → Contains ASCII-encoded text.  -------------------> interesting
- **/underground** → "You’ve found the wrong turn! The flag’s waiting somewhere else!"
- **/secret_path** → Contains Morse code.  ----------------------> interesting
- **/hidden_flag** → "The deeper you go, the more you find... but sometimes you’ll need to dig a little."
- **/buried_flag** → "You’re getting warmer... or maybe colder? Let’s see what the next path has in store!"
- **/encrypted** → "Sometimes the answers are hidden in plain sight. Or, in this case, styled just right."

### Finding the First Part of the Flag

In the `/buried` directory, I found the following text:

```
49 115 116 32 80 97 114 116 32 111 102 32 116 104 101 32 70 108 97 103 32 105 115 32 58 32 65 67 69 67 84 70 123 49 110 102 49 108 55 114 52 55 49 110 103 95 55 104 51 95 53 121 53 55 51 109 95 32
```

This looked like ASCII values. Using an [ASCII to String converter](https://onlinestringtools.com/convert-ascii-to-string), I decoded it to:

```
1st Part of the Flag is : ACECTF{1nf1l7r471ng_7h3_5y573m_
```

### Finding the Second Part of the Flag

The `/secret_path` directory contained Morse code:

```
..--- -. -..
.--. .- .-. -
--- ..-.
- .... .
..-. .-.. .- --.
.. ...
---...
.---- ..... ..--.- ...-- ....- ..... -.-- ..--.- .-- .... ...-- -. ..--.- -.-- ----- ..- ..--.- -.- -. ----- .-- ..--.- .-- .... ...-- .-. ...-- ..--.-
```

Using the [CyberChef Morse code decoder](https://gchq.github.io/CyberChef/#recipe=From_Morse_Code('Space','Line%20feed')&oenc=65001), I decoded it to:

```
2ND PART OF THE FLAG IS : 15_345Y_WH3N_Y0U_KN0W_WH3R3_
```

Converting it to lowercase, I obtained:

```
2nd part of the flag is : 15_345y_wh3n_y0u_kn0w_wh3r3_
```

### Finding the Final Part of the Flag

The `/encrypted` directory contained a CSS file with the following content:

```
#flag {
    display: none;
    content: "bC5 !2CE @7 E96 u=28 :D i f9b0db4CbEd0cCb03FC`b5N"; 
}
```

This looked like a ROT cipher. I tried both [ROT13](https://en.wikipedia.org/wiki/ROT13) and [ROT47](https://en.wikipedia.org/wiki/ROT47). Decoding it revealed:

```
3rd Part of the Flag is : 7h3_53cr3t5_4r3_bur13d}
```

## Final Flag

Combining all three parts:

```
ACECTF{1nf1l7r471ng_7h3_5y573m_15_345y_wh3n_y0u_kn0w_wh3r3_7h3_53cr3t5_4r3_bur13d}
```

## References

- [ASCII Table](https://en.wikipedia.org/wiki/ASCII)
- [Morse Code](https://en.wikipedia.org/wiki/Morse_code)
- [ROT13 Cipher](https://en.wikipedia.org/wiki/ROT13)

This challenge was a great example of web enumeration, encoding techniques, and cryptographic analysis. Hope this write-up helps others in tackling similar CTF challenges!

