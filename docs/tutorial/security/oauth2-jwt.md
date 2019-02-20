Now that we have all the security flow, let's make the application actually secure, using JWT tokens and secure password hashing.

This code is something you can actually use in your application, save the password hashes in your database, etc.

We are going to start from where we left in the previous chapter and increment it.

## About JWT

JWT means "JSON Web Tokens".

It's a standard to codify a JSON object in a long string.

It is not encrypted, so, anyone could recover the information from the contents.

But it's signed. So, when you receive a token that you emitted, you can verify that you actually emitted it.

That way, you can create a token with an expiration of, let's say, 1 week, and then, after a week, when the user comes back with the token, you know he's still signed into your system.

And after a week, the token will be expired. And if the user (or a third party) tried to modify the token to change the expiration, you would be able to discover it, because the signature would not match.

If you want to play with JWT tokens and see how they work, check <a href="https://jwt.io/" target="_blank">https://jwt.io</a>.

## Install `PyJWT`

We need to install `PyJWT` to generate and verity the JWT tokens in Python:

```bash
pip install pyjwt
```

## Password hashing

"Hashing" means converting some content (a password in this case) into a sequence of bytes (just a string) that look like gibberish.

Whenever you pass exactly the same content (exactly the same password) you get exactly the same gibberish.

But you cannot convert from the gibberish back to the password.

### What for?

If your database is stolen, the thief won't have your users' plaintext passwords, only the hashes.

So, the thief won't be able to try to use that password in another system (as many users use the same password everywhere, this would be dangerous).

## Install `passlib`

PassLib is a great Python package to handle password hashes.

It supports many secure hashing algorithms, and utilities to work with them.

The recommended algorithm is "Bcrypt".

So, install PassLib with Bcrypt:

```bash
pip install passlib[bcrypt]
```

!!! tip
    With `passlib`, you could even configure it to be able to read passwords created by **Django** (among many others).

    So, you would be able to, for example, share the same data from a Django application in a database with a FastAPI application. Or gradually migrate a Django application using the same database.


## Hash and verify the passwords

Import the tools we need from `passlib`.

Create a PassLib "context". This is what will be used to hash and verify passwords.

!!! tip
    The PassLib context also has functionality to use different hashing algorithms, deprecate old ones, but allow verifying them, etc.

    For example, you could use it to read and verify passwords generated by another system (like Django) but hash any new passwords with a different algorithm like Bcrypt.

    And be compatible with all of them at the same time.

Create a utility function to hash a password coming from the user.

And another utility to verify if a received password matches the hash stored.

And another one to authenticate and return a user.

```Python hl_lines="7 50 57 58 61 62 71 72 73 74 75 76 77"
{!./src/security/tutorial004.py!}
```

!!! note
    If you check the new (fake) database `fake_users_db`, you will see how the hashed password looks like now: `"$2b$12$EixZaYVK1fsbw1ZfbX3OXePaWxn96p36WQoeG6Lruj3vjPGga31lW"`.

## Handle JWT tokens

Import the modules installed.

Create a random secret key that will be used to sign the JWT tokens.

To generate a secure random secret, key use the command:

```bash
openssl rand -hex 32
```

And copy the output to the variable `SECRET_KEY` (don't use the one in the example).

Create a variable `ALGORITHM` with the algorithm used to sign the JWT token and set it to `"HS256"`.

And another one for the `TOKEN_SUBJECT`, and set it to, for example, `"access"`.

Create a variable for the expiration of the token.

Define a Pydantic Model that will be used in the token endpoint for the response.

Create a utility function to generate a new access token.

```Python hl_lines="3 6 13 14 15 16 30 31 32 80 81 82 83 84 85 86 87 88"
{!./src/security/tutorial004.py!}
```

## Update the dependencies

Update `get_current_user` to receive the same token as before, but this time, using JWT tokens.

Decode the received token, verify it, and return the current user.

If the token is invalid, return an HTTP error right away.

```Python hl_lines="91 92 93 94 95 96 97 98 99 100"
{!./src/security/tutorial004.py!}
```

## Update the `/token` path operation

Create a `timedelta` with the expiration time of the token.

Create a real JWT access token and return it.

```Python hl_lines="114 115 116 117 118"
{!./src/security/tutorial004.py!}
```

## Check it

Run the server and go to the docs: <a href="http://127.0.0.1:8000/docs" target="_blank">http://127.0.0.1:8000/docs</a>.

You'll see the user interface like:

<img src="/img/tutorial/security/image07.png">

Authorize the application the same way as before.

Using the credentials:

Username: `johndoe`
Password: `secret`

!!! check
    Notice that nowhere in the code is the plaintext password "`secret`", we only have the hashed version.

<img src="/img/tutorial/security/image08.png">

Call the endpoint `/users/me`, you will get the response as:

```JSON
{
  "username": "johndoe",
  "email": "johndoe@example.com",
  "full_name": "John Doe",
  "disabled": false
}
```

<img src="/img/tutorial/security/image09.png">

If you open the developer tools, you could see how the data sent and received is just the token, the password is only sent in the first request to authenticate the user:

<img src="/img/tutorial/security/image10.png">

!!! note
    Notice the header `Authorization`, with a value that starts with `Bearer `.

## Advanced usage with `scopes`

We didn't use it in this example, but `Security` can receive a parameter `scopes`, as a list of strings.

It would describe the scopes required for a specific path operation, as different path operations might require different security scopes, even while using the same `OAuth2PasswordBearer` (or any of the other tools).

This only applies to OAuth2, and it might be, more or less, an advanced feature, but it is there, if you need to use it.

## Recap

This concludes our tour for the security features of **FastAPI**.

In almost any framework handling the security becomes a rather complex subject quite quickly.

Many packages that simplify it a lot have to make many compromises with the data model, database, and available features. And some of these packages that simplify things too much actually have security flaws underneath.

---

**FastAPI** doesn't make any compromise with any database, data model or tool.

It gives you all the flexibility to chose the ones that fit your project the best.

And you can use directly many well maintained and widely used packages like `passlib` and `pyjwt`, because **FastAPI** doesn't require any complex mechanisms to integrate external packages.

But it provides you the tools to simplify the process as much as possible without compromising flexibility, robustness or security.

And you can use secure, standard protocols like OAuth2 in a relatively simple way.