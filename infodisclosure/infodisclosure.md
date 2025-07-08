# Information Disclosure
Information Disclosure refers to the unauthorized exposure or leakage of sensitive information, such as personal data or system details, to unintended parties.

This example demonstrates information disclosure by injecting malicious query objects to a NoSQL database.

## What is the vulnerability?

Consider the following code snippet in **insecure.ts**:

```
app.get('/userinfo', async (req: Request, res: Response) => {
  const { username } = req.query;
  console.log(username);

  // Vulnerable code: Directly using user-provided values in the query
  const user = await User.findOne({ username: username as string }).exec();

  if (user) {
    res.send(`User: ${user}`);
  } else {
    res.status(401).send('Invalid username or password');
  }
});
```

The MongoDB query `User.findOne()` depends on a variable whose value is provided by an incoming request. If this value is a valid MongoDB filter condition then the query will potentially be executed by an attacker controlled filter condition thereby disclosing information to an unauthorized entity.

There are several ways to prevent this vulnerability. If the data stored is sensitive then we must encrypt it to reduce the damage in the event of a disclosure. Or else we can ensure that all queries use only sanitized inputs. The latter approach is encoded in **secure.ts**. 

## For you to do

Steps to reproduce:

1. Install all dependencies

    `$ npm install`

2. Insert test data in the MongoDB database. Make sure the mongod is up and running by typing the `mongosh` command in the termainal. If mongod process is up then you will see that the connection was successful. Command to insert test data:

    `$ npx ts-node insert-test-users.ts`

This will create a database in MongoDB called __infodisclosure__. Verify its presence by connecting with mongosh and running the command `show dbs;`.

2. Start the **insecure.ts** server

    `$ npx ts-node insecure.ts`

3. In the browser, pretend to be a hacker and type a malicious request

    ```
        http://localhost:3000/userinfo?username[$ne]=
    ```

4. Do you see user information being displayed despite the malicious request not having a valid username in the request?

5. Repeat the steps with **secure.ts**. Is the attack still successful? Why?