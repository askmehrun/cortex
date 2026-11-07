# Overleaf Password Reset (MongoDB)

Reset the admin password of a self-hosted Overleaf (Ayakaleaf Pro) by writing a new bcrypt hash directly into MongoDB, when you have VM access but lost the credentials.

## When to Use
- The admin password is lost and the normal password-reset email flow doesn't work.
- You have shell access to the host and the mongo container.

## How
1. Confirm the database name:
   ```bash
   docker exec sharelatex env | grep -i mongo → mongodb://mongo/sharelatex
   ```
2. Find the admin by email and copy the `_id`:
   ```bash
   docker exec mongo mongosh sharelatex --eval 'db.users.findOne({email:"..."}, {email:1,_id:1,role:1})'
   ```
   The collection is `users` (plural), not `user`.
3. Generate a bcrypt hash on the host (Python, 12 rounds):
   ```bash
   python3 -c "import bcrypt; print(bcrypt.hashpw(b'NEWPASS', bcrypt.gensalt(12)).decode())"
   ```
4. Write it into MongoDB:
   ```bash
   docker exec mongo mongosh sharelatex --eval '
     db.users.updateOne({_id: ObjectId("...")},
       {$set:{hashedPassword:"HASH", role:"admin", emailVerifiedAt: new Date(), resetToken:null}});'
   ```
5. Restart if login is stale:
   ```bash
   sudo systemctl restart docker
   ```

## Gotchas
- The field is `hashedPassword` (not `password`); the value is a bcrypt 12-round hash.
- bcrypt truncates at 72 characters.
- `role` is often an empty string — set it to `"admin"` explicitly.
- Never generate the hash inside bash with `!!` — history expansion corrupts the password. Use Python or a file.

## Rule
The hash must be generated outside bash; a shell history expansion invalidates it.
