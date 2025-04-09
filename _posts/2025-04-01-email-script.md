---
layout: post
title:  "Yahoo Mail Alerts!"
date:   2025-04-01
categories: automation
---

Due to the flood of emails from Amazon I created a special folder in my inbox with filters that collect all Amazon emails in one place. Unfortunately, the delivery notifications from Amazon Locker ended up there too, and I missed a lot of them since I didn't check the folder often. 

### Here's the plan I came up with: 

- Periodically check your E-Mail for new messages (like USPS alerts). 

- Filter based on subject/content. 

- Send yourself a notification or summary. 

While searching for a solution, I stumbled across a helpful blog post on Hacker News that was about Gmail and Google Keep. I wanted to see if this could work with other email providers and to-do apps, so I turned to ChatGPT for help. 

I asked ChatGPT if it was possible to create a similar setup for my email provider, and it gave me a detailed guide on how to do it using IMAP and OAuth-based API access, along with a server-side script in Python or Node.js. This helped me set up the email filtering and checking part. 

Next, I needed a way to notify myself about the package deliveries. Since I use Todoist, I asked ChatGPT if we could add tasks to my Todoist list. ChatGPT provided step-by-step instructions on how to set up an API token and extended the script to add tasks to Todoist. 

And there you have it! A fun and easy way to make sure you never miss those important Amazon Locker delivery notifications again. 

```python 
import imaplib
import email
from email.header import decode_header
import requests

# E-Mail IMAP settings
imap_host = ""
username = ""
password = ""

# Todoist API
todoist_token = "your_todoist_api_token"
todoist_api_url = "https://api.todoist.com/rest/v2/tasks"
# Optional: Set a project ID or leave it blank for Inbox
todoist_project_id = None  # or 'your_project_id'

# Connect to E-Mail IMAP
mail = imaplib.IMAP4_SSL(imap_host)
mail.login(username, password)
mail.select("inbox")

# Search for unread USPS emails
status, messages = mail.search(None, '(UNSEEN SUBJECT "USPS")')
for num in messages[0].split():
    status, data = mail.fetch(num, '(RFC822)')
    msg = email.message_from_bytes(data[0][1])
    subject = decode_header(msg["Subject"])[0][0]
    if isinstance(subject, bytes):
        subject = subject.decode()
    print("Found USPS Mail:", subject)

    # Create task in Todoist
    task_data = {
        "content": f"📬 USPS Alert: {subject}",
        "due_string": "today",
    }

    if todoist_project_id:
        task_data["project_id"] = todoist_project_id

    headers = {
        "Authorization": f"Bearer {todoist_token}",
        "Content-Type": "application/json",
    }

    response = requests.post(todoist_api_url, json=task_data, headers=headers)

    if response.status_code == 200 or response.status_code == 204:
        print("✅ Task added to Todoist")
    else:
        print("❌ Failed to add task", response.text)

mail.logout()
```