# Spoofing
A spoofing attack happens when an attacker successfully identifies as another by falsifying or stealing data to gain unauthorized access.

Spoofing can manifest in several ways. This example demonstrates how a spoofing attack can be carried out in a web server using sessions and cookies.

## Background Concepts

### Cookies and Sessions

To understand the spoofing threat in this example, we have to understand the concept of cookies and sessions, which were developed to address the stateless nature of the HTTP protocol. Simply put the HTTP protocol defines the rules for sending and receiving requests and responses between a client program and a server program. As HTTP is stateless, every request is treated as new request by the server. This poses a problem for the client program, especially if the client needs to make itself known to the server. One way to do this is to create a unique identifier for the client and the server to maintain this identifier in its state. This unique identifier is known as a session ID and is created when the client authenticates itself to the server using some form of credentials (e.g., username/password). The session ID is typically sent by the server to the client. When the client receives it, it saves the session ID in a cookie and sends it with every request to the server.

### How Sessions and Cookies enable Spoofing?

The session ID is critical. If a malicious attacker gets access to the session ID, they can masquerade as the client program to the server. All they have to do is send the same session ID to the server. When this happens its called **session hijacking**, an example of how a spoofing attack manifests. 

Cookies are browser mechanisms used by a client program to store small pieces of information. Information stored in cookies can be programatically accessed by any client running in the browser. This implies that if critical information such as a session ID is stored in a cookie then a malicious program can read it and also change it. This is an example of a cookie injection attack, another attack method (vector) through which spoofing threats are exploited.

## What is the vulnerability?

Consider the middleware in **insecure.ts**:

```
app.use(
  session({
    secret: "SOMESECRET",
    cookie: { httpOnly: false },
    resave: false,
    saveUninitialized: false,
  })
);
```

This indicates that the server program is configured to create a session ID, create a hash of the session ID with the secret, and store it in a cookie in the client program that sent the request. Once the cookie is set in the browser, the client program is going to send the hashed session ID with every request. The middleware is going to intercept every such request and verify if the session ID in the request in the same as the session ID that was generated. If a request does not have the session ID in its cookie then a new session is created. 

There are several problems in the middleware:

1. First, note how the cookie is configured with the `httpOnly` flag set to *false*, which implies that the cookie can be accessed from any client program with access to the browser DOM. A malicious program can do two things -- (1) read the session ID and use it later to hijack the session, or (2) change the session ID and cause the client requests to be rejected causing a denial of service. 

2. The server is configured to accept session ID cookies from any client. Browser cookies have a default behavior -- once set they will be sent with every request to the server. This implies that once set, a malicious client program will be able to send the cookie containing the session ID. The client program has to only convince an unsuspecting user to load the program in the browser and click on some links that will send a malicious request with the cookie to the server. This is called a cross-site request forgery attack because the cookies will be sent from a client not in the same domain as the server.

3. The secret is hard-coded in plain text. Malicious actors with access to the code can leak the secret, which can then be used to guess the session ID.

These potential vulnrabilities have been fixed in **secure.ts**. Look at the code to see how.

## For you to do

Steps to reproduce the vulnerability:

1. Install dependencies

    `$ npx install`

2. Start the **insecure.ts** server

    `npx ts-node insecure.ts`

3. Start the malicious server **mal.ts**

    `npx ts-node mal.ts`

4. Open __http://localhost:8000__ in a browser, type `Admin` in name field and Submit.

5. Open the __Application__ tab in the Browser's inspect pane. Find the __Cookies__ under __Storage__. You should see a __connect.sid__ cookie being set.

6. Open the HTML file __mal-steal-cookie.html__ file in the same browser (different tab). Open inspect and view the console.

7. Click the link in the HTML file. Do you see the cookie being stolen in the console? What kind of spoofing attack is this?

8. Open the HTML file __mal-csrf.html__ in the same browser (different tab). What do you see if the user has not logged out of **insecure.ts**? What do you see if the user has logged out? What can you conclude from your observation.

If the user is still logged in with name 'Admin' and when the __mal-csrf.html__ file loads in the browser, you should expect to see an "operation successful" message. If you do not see it, delete the cookie with the session ID from the browser, login with 'Admin' and try loading the HTML in the browser again.