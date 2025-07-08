# Privilege Escalation
Unauthorized elevation of privileges, granting access to resources or actions beyond the intended level of authorization.

Few reasons for this threat are - implementation weaknesses, absence of authorization, exposing services that do not need to be.

This example demonstrates privilege escalation due to absence of meaningful authorization.

## What is the vulnerability?

The web server in **insecure.ts** has a POST service that allows users with `admin` role to update the role of existing users. The following code snippet illustrates this:

```
app.post('/update-role', (req: Request, res: Response) => {
  const { userId, newRole } = req.body;

  // Simulated authentication (insecure)
  const user = users.find(u => u.id === Number(userId));
  if (!user) {
    return res.status(404).json({ error: 'User not found' });
  }

  // Simulated authorization (insecure)
  if (user.role !== 'admin') {
    return res.status(403).json({ error: 'Unauthorized' });
  }

  // Update user role (vulnerable to privilege escalation)
  user.role = newRole;
  res.json({ message: 'User role updated successfully' });
});
```

The user role conditional check is based on the incoming request, which is inherently untrusted. This implies that an existing user with any role can masquerade as admin (elevate their privilege) and change roles of existing users. 

The secure version in **secure.ts** addresses this issue by retrieving a user's role from correctly authenticated client using the concept of sesisons. This implies that only an admin can change user roles as they will have to first authenticate as an admin to obtain a valid session token/ID. As long as the sessions (cookies) are correctly configured, this approach will prevent privilege escalation.

## For you to do

Steps to reproduce:

1. Install all dependencies

    `$ npm install`

2. Start the **insecure.ts** server

    `$ npx ts-node insecure.ts`

3. In the browser, send a GET request

    ```
        http://localhost:3000/send-form
    ```

4. Try different UserIds and see which one gives you authorized access to change the role of that user.

5. Repeat the steps with **secure.ts**. Why does this version prevent privilege escalation?