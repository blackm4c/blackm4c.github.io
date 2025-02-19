---
title: HTB - Web Challange - SpookTastic
date: 2025-02-19 20:00:00 +0530
categories: [HackTheBox, Challanges]
tags: [webchall,xss,sourcecodereview]     # TAG names should always be lowercase
description: Review the source code to identify vulnerabilities that may allow bypassing the XSS blocklist.
toc: true
comments: false
# pin: true
# published: false
---

## Recon

Once I started the instance, I opened the website using its IP address for an initial analysis. While exploring the page, I found an email input field at the bottom for newsletter registration. This field sends the user input to the server.

![Desktop View](/assets/img/htb/chall/web/spooktastic/input_field.png){: .w-75 .shadow .rounded-15 w='1212' h='668'  }
_Input field_

Checking the page source, I discovered JavaScript code containing an API endpoint: `/api/register`. Since the challenge files were included, I proceeded to analyze the source code.

![Desktop View](/assets/img/htb/chall/web/spooktastic/javascript_code.png){: .w-75 .shadow .rounded-15 w='1212' h='668' }
_Javascript code_

---

### Source Code Analysis

After unzipping the challenge files, I found a Docker setup for a Flask app. I then opened the `app.py` file to analyze its logic.

### Flask App Code

```python
import random, string
from flask import Flask, request, render_template, abort
from flask_socketio import SocketIO
from threading import Thread

app = Flask(__name__)

socketio = SocketIO(app)

registered_emails, socket_clients = [], {}

generate = lambda x: "".join([random.choice(string.hexdigits) for _ in range(x)])
BOT_TOKEN = generate(16)

def blacklist_pass(email):
    email = email.lower()

    if "script" in email:
        return False

    return True


def send_flag(user_ip):
    for id, ip in socket_clients.items():
        if ip == user_ip:
            socketio.emit("flag", {"flag": open("flag.txt").read()}, room=id)


def start_bot(user_ip):
    from selenium import webdriver
    from selenium.webdriver.chrome.options import Options
    from selenium.webdriver.chrome.service import Service
    from selenium.webdriver.support.ui import WebDriverWait
    from selenium.webdriver.support import expected_conditions as EC

    host, port = "localhost", 1337
    HOST = f"http://{host}:{port}"

    options = Options()
    options.add_argument("--headless")
    options.add_argument("--no-sandbox")
    options.add_argument("--disable-dev-shm-usage")
    options.add_argument("--disable-infobars")
    options.add_argument("--disable-background-networking")
    options.add_argument("--disable-default-apps")
    options.add_argument("--disable-extensions")
    options.add_argument("--disable-gpu")
    options.add_argument("--disable-sync")
    options.add_argument("--disable-translate")
    options.add_argument("--hide-scrollbars")
    options.add_argument("--metrics-recording-only")
    options.add_argument("--mute-audio")
    options.add_argument("--no-first-run")
    options.add_argument("--dns-prefetch-disable")
    options.add_argument("--safebrowsing-disable-auto-update")
    options.add_argument("--media-cache-size=1")
    options.add_argument("--disk-cache-size=1")
    options.add_argument("--user-agent=HTB/1.0")

    service = Service(executable_path="/usr/bin/chromedriver")
    browser = webdriver.Chrome(service=service, options=options)

    try:
        browser.get(f"{HOST}/bot?token={BOT_TOKEN}")

        WebDriverWait(browser, 3).until(EC.alert_is_present())

        alert = browser.switch_to.alert
        alert.accept()
        send_flag(user_ip)
    except Exception as e:
        pass
    finally:
        registered_emails.clear()
        browser.quit()


@app.route("/")
def index():
    return render_template("index.html")


@app.route("/api/register", methods=["POST"])
def register():
    if not request.is_json or not request.json["email"]:
        return abort(400)
    
    if not blacklist_pass(request.json["email"]):
        return abort(401)

    registered_emails.append(request.json["email"])
    Thread(target=start_bot, args=(request.remote_addr,)).start()
    return {"success": True}


@app.route("/bot")
def bot():
    if request.args.get("token", "") != BOT_TOKEN:
        return abort(404)
    return render_template("bot.html", emails=registered_emails)


@socketio.on("connect")
def on_connect():
    socket_clients[request.sid] = request.remote_addr


@socketio.on("disconnect")
def on_disconnect():
    del socket_clients[request.sid]


if __name__ == "__main__":
    app.run(host="0.0.0.0", port=1337, debug=False)

```

### Analyzing `/api/register` Endpoint
The `/api/register` route handles user email registration.
```python
@app.route("/api/register", methods=["POST"])
def register():
    if not request.is_json or not request.json["email"]:  # Check if the request contains JSON data with an email
        return abort(400)                                       
    
    if not blacklist_pass(request.json["email"]):  # Validate user input with blacklist_pass function
        return abort(401)

    registered_emails.append(request.json["email"])  # If valid, add the email to registered_emails
    Thread(target=start_bot, args=(request.remote_addr,)).start()  # Start a new bot process
    return {"success": True}
```
{: .nolineno }
Steps of this function:

- It checks if the request contains JSON data with an email. If not, it returns a 400 status.
- The email is then passed to the blacklist_pass function for validation.
- If the email is valid, it is added to registered_emails.
- A new thread starts the `start_bot` function, passing the user's IP address as an argument.

Analyzing `blacklist_pass` Function

```python
def blacklist_pass(email):
    email = email.lower()

    if "script" in email:
        return False

    return True
```
{: .nolineno }
This function tries to prevent XSS by blocking emails that contain the word "script". However, this check is weak because it only filters "script" instead of validating all possible XSS payloads. This makes it a vulnerable point for exploitation.

Understanding `start_bot` Function
```python
def start_bot(user_ip):
    from selenium import webdriver
    from selenium.webdriver.chrome.options import Options
    from selenium.webdriver.chrome.service import Service
    from selenium.webdriver.support.ui import WebDriverWait
    from selenium.webdriver.support import expected_conditions as EC

    host, port = "localhost", 1337
    HOST = f"http://{host}:{port}"

    options = Options()

    options.add_argument("--headless")
    options.add_argument("--no-sandbox")
    options.add_argument("--disable-dev-shm-usage")
    options.add_argument("--disable-infobars")
    options.add_argument("--disable-background-networking")
    options.add_argument("--disable-default-apps")
    options.add_argument("--disable-extensions")
    options.add_argument("--disable-gpu")
    options.add_argument("--disable-sync")
    options.add_argument("--disable-translate")
    options.add_argument("--hide-scrollbars")
    options.add_argument("--metrics-recording-only")
    options.add_argument("--mute-audio")
    options.add_argument("--no-first-run")
    options.add_argument("--dns-prefetch-disable")
    options.add_argument("--safebrowsing-disable-auto-update")
    options.add_argument("--media-cache-size=1")
    options.add_argument("--disk-cache-size=1")
    options.add_argument("--user-agent=HTB/1.0")

    service = Service(executable_path="/usr/bin/chromedriver")
    browser = webdriver.Chrome(service=service, options=options)

    try:
        browser.get(f"{HOST}/bot?token={BOT_TOKEN}")

        WebDriverWait(browser, 3).until(EC.alert_is_present())

        alert = browser.switch_to.alert
        alert.accept()
        send_flag(user_ip)
    except Exception as e:
        pass
    finally:
        registered_emails.clear()
        browser.quit()
```
{: .nolineno }

This function does the following:

- Starts a headless Selenium Chrome browser.
- Opens the `/bot` page with a secret bot token.
- Waits for an alert pop-up on the webpage.
- If an alert appears, it calls `send_flag(user_ip)`.

### XSS Vulnerability in `bot.html`

The `bot.html` template renders registered emails without sanitization:

```html
 {for email in emails} 
    <span>{{ email|safe }}</span><br/>
 {endfor} 
```
{: .nolineno }
Since the email input is directly inserted into the HTML without escaping, any JavaScript code inside the email will execute.

The `send_flag` function sends the user's flag through an open socket connection if their IP matches a connected client.

```python
def send_flag(user_ip):
    for id, ip in socket_clients.items():
        if ip == user_ip:
            socketio.emit("flag", {"flag": open("flag.txt").read()}, room=id)
```
{: .nolineno }
This allows the bot to send the flag only to the attacker’s session upon successful XSS execution.

## Exploitation

The blacklist only blocks the word "script". This means we can bypass the filter using a different XSS payload.

### Bypassing the Blacklist
Instead of using a `<script>` tag, we can use an `<img>` tag with an onerror event:

`<img src=0 onerror=alert(1)>`

Attack Steps:
- Submit the payload as an email: `<img src=0 onerror=alert(1)>`
- The input passes the blacklist_pass check since it does not contain "script".
- The bot processes the input and renders it in bot.html, where it executes as JavaScript.
- The alert pops up, and the bot calls send_flag(user_ip), sending us the flag.

## Conclusion
This challenge demonstrates how weak blacklist-based XSS protection can be bypassed. Instead of blocking specific words like "script", proper input validation and output encoding should be used to prevent XSS attacks.
