# Repudiation

Non-repudiation refers to the ability of systems to hold people accountable to their actions. E.g., Malicious user deletes sensitive information, but the system cannot track down the user. 

Systems can be non-repudiable for several reasons ranging from insufficient logging, logs vulnerable to tampering, lack of audit policies, etc. The example demonstrates a vulnerability that can lead to non-repudiation by malicious users attempting to access the services provided by a server without leaving trace.

## What is the vulnerability?

The web server in **insecure.ts** defines a POST and GET service to send and receive messages, respectively. However, it does not keep track of the requests received or the responses sent. The secure version in **secure.ts** addresses this issue by defining a middleware for logging. The middleware intercepts each incoming request and logs the IP address and the time the request was made. This ensures that in the event of a security attack, the logs can be inspected to trace the potential attacker.

## For you to do

Steps to reproduce:

1. Install all dependencies

    `$ npm install`

2. Run the server __insecure.ts__.

3. Pretend to be a malicous user and interact with the services by sending requests from the browser.

4. Do you think your actions can be repudiated?