# Denial-of-Service (DoS)

Attackers can prevent access to a service by overwhelming it with requests or by exploiting a vulnerability to crash it.

Reasons for a DoS vulneability can range from unnecessary exposure to services, to a lack of rate limiting, to a failure to handle high traffic loads.

This example demonstrates a DoS vulnerabilities and how it can be exploited.

## What is the vulnerability?

Consider the GET route function in the web server **insecure.ts**:

```
app.get('/userinfo', async (req: Request, res: Response) => {
  const { id } = req.query;

  const uid = id as string;

  const user = await User.findOne({ _id: uid }).exec();

  if (user) {
    res.send(`User: ${user}`);
  } else {
    res.status(401).send('User not found');
  }
});
```

The `id` used in the MongoDB query `User.findOne` is expected to be a valid MongoDB query. However, since there is no error handling, if the incoming request has a an id string that is an invalid MongoDB ID then the server will crash. A malicious actor can spam the server with such invalid ID strings, continuously crashing it, and deny the service to legitimate users.

In the secure version **secure.ts** this vulnerability is prevented by adding proper exception handling and by adding a rate lmiter, which limits every client to sending a specific number of requests per unit time.

## For you to do

Steps to reproduce:

1. Install all dependencies

    `$ npm install`

2. Ignore if you have already done this once. Insert test data in the MongoDB database. Make sure the mongod is up and running by typing the `mongosh` command in the termainal. If mongod process is up then you will see that the connection was successful. Command to insert test data:

    `$ npx ts-node insert-test-users.ts`

This will create a database in MongoDB called __infodisclosure__. Verify its presence by connecting with mongosh and running the command `show dbs;`.

2. Start the **insecure.ts** server

    `$ npx ts-node insecure.ts`

3. In the browser, pretend to be a hacker and type a malicious request

    ```
        http://localhost:3000/userinfo?id[$ne]=
    ```

4. Do you see the server crashing?

5. Repeat the steps with **secure.ts**. Is the threat removed? How?