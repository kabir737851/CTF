## Problem

<img width="923" height="794" alt="image" src="https://github.com/user-attachments/assets/92053162-1965-491b-8710-b4422dd82dd9" />

#Solution

We start a challenge instance and see a small website.
The challenge hints that the site may be vulnerable to SSTI (Server-Side Template Injection).

<img width="1239" height="468" alt="image" src="https://github.com/user-attachments/assets/2220803c-4d49-4893-9dab-2a94846eb92b" />

Template engines create dynamic pages. If user input is passed directly into a template, attackers can inject code into it. That’s what SSTI is.

Before trying to attack the website, we first need to figure out which template engine it uses. This is important because different template engines need different types of payloads. To understand this better, I used a helpful diagram from https://www.cobalt.io/blog/a-pentesters-guide-to-server-side-template-injection-ssti that explains which syntax belongs to which engine.

<img width="821" height="520" alt="image" src="https://github.com/user-attachments/assets/ecf0635d-c7d5-4b7a-9585-51d1d643dabe" />

So I first tried the payload ${7*7}. The website just showed ${7*7} as plain text, which means it didn’t execute anything.
Based on the diagram, I then tested {{7*7}}. This time, the website actually calculated the result and showed 49.

<img width="1047" height="361" alt="image" src="https://github.com/user-attachments/assets/6980bb4a-094b-4089-a2b7-e9f5e4d7ce43" />

<img width="912" height="386" alt="image" src="https://github.com/user-attachments/assets/0e7410c7-a983-4cfa-b493-82edb5501dd4" />

I also tried {{7*'7'}}, and that was evaluated too.
This confirms that the site is processing these expressions, showing that an SSTI vulnerability exists.

<img width="648" height="247" alt="image" src="https://github.com/user-attachments/assets/bd5fbb2c-2c24-4e0d-b85a-4a9bd99b0fac" />

<img width="693" height="255" alt="image" src="https://github.com/user-attachments/assets/38efa254-14ae-41e8-86b2-f564cdddd312" />

Since our inputs were being processed, it proves the website is vulnerable to SSTI.
This also tells us the site is probably using either Jinja2 or Twig, which are both Python-based template engines.
That means our exploit payloads should also use Python.

If we try something like {{request}}, we can see the request object, which gives us access to even more useful information inside the application.

<img width="694" height="258" alt="image" src="https://github.com/user-attachments/assets/8707d8ab-a8a1-4f2f-a7a4-ee86e0d088b2" />

<img width="1895" height="546" alt="image" src="https://github.com/user-attachments/assets/fa02e220-5061-4bdf-ab49-ba531a3ecda8" />

From this point, we can access almost everything inside the application by using Python’s special https://www.geeksforgeeks.org/python/dunder-magic-methods-python/.
While researching Python SSTI attacks, I found a helpful https://onsecurity.io/article/server-side-template-injection-with-jinja2/ article that explains different techniques for this.
For example, the payload below lets us run the ls command on the server’s current directory:

```
{{request.application.__globals__.__builtins__.__import__('os').popen('cat flag').read()}}
```
<img width="658" height="220" alt="image" src="https://github.com/user-attachments/assets/0831099e-7026-499b-a6c0-f582fcccf60a" />

<img width="1580" height="355" alt="image" src="https://github.com/user-attachments/assets/9437c324-8496-47e2-9679-694e7f60f3b9" />

And of course, we want to read the flag. So we can modify the previous payload to run cat flag instead.
```
{{request.application.__globals__.__builtins__.__import__('os').popen('cat flag').read()}}
```
<img width="675" height="249" alt="image" src="https://github.com/user-attachments/assets/48f13db8-67f9-496f-83d5-01887eaa20a4" />

<img width="1857" height="241" alt="image" src="https://github.com/user-attachments/assets/13c88a41-35e7-41ae-bf79-85a391e99546" />











