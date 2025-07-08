# Tampering
Attackers can change or tamper with the application’s inputs to cause unintended behavior.

Tampering can occurs in a number of ways. This example demonstrates tampering through script injection.

## What is the vulnerability?

Consider the following code snippet in **insecure.ts**:

```
app.post("/register", (req: Request, res: Response) => {
  req.session.user = req.body.name.trim();
  res.send(`<p>Thank you</p> <a href="/">Back home</a>`);
});
```

Note the server session object's `user` property is being populated with with a string from an incoming request (from some client). This incoming string is inherently untrustworthy as it could be sent by a malicious program. Particularly if the incoming string is a script and the server program uses (executes) the `user` property then script will end up being executed inadvertently. This is called a script injection attack. 

The most effective way to prevent tampering via script injection is to sanitize the incoming inputs from potentially untrustworthy sources. Sanitizing inputs means that the program should have a whitelist of inputs it will accept and reject everything else. The server **secure.ts** does exactly this.

## For you to do

Steps to reproduce:

1. Install all dependencies

    `npm install`

2. Start the **insecure.ts** server

    `npx ts-node insecure.ts`

3. In the browser, type a potentially malicious script in the name field of the form

    ```
        <script> document.body.innerHTML = "<a href='https://google.com'> Gotcha </a>"</script>
    ```

4. Do you see the potentially malicious hyperlink being injected into the form? Repeat the same steps with **secure.ts**. Do you see any difference. Why?