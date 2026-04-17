## Doc
```
JSON web token
```
### Structure
```
JWT has three parts separated by dots
Header: tells the algorithm used (like HS256)
Payload: stores user data (user ID, email, expiration time)
Signature: a secure hash to prevent tampering

example:
eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJ1c2VySWQiOjEsImVtYWwiOiJhYXJvbkBleGFtcGxlLmNvbSIsImV4cCI6MTc0NDAwMDAwMH0.XbPxDbTZJzGyj_rs2ZQk8xN0n4LdbA0bLdH0Rnl4Vf8
```
### example
```
header:
{"alg":"HS256","typ":"JWT"}
base46:eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9


payload:
{ "userId": 123, "email": "aaron@example.com", "name": "Aaron Guo", "exp": 1746000000 }
base46:eyJ1c2VySWQiOjEyMywiZW1haWwiOiJhYXJvbkBleGFtcGxlLmNvbSIsIm5hbWUiOiJBYXJvbiBHdW8iLCJleHAiOjE3NDYwMDAwMDB9

signature: encrypt header + payload with a secret key only your server knows. It ensures the token cannot be modified or faked.

```