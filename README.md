----------------------------------
BCrypt and Authentication
----------------------------------
- bcrypt and secure passwords
- cookies and sessions
- implementation notes

----------------------------------
Bcrypt Secure passwords
----------------------------------
- Cant store pwds in the clear
- We dont want to know ur password

- Bcrypt is an algorithm
We want a function where given a string, we can tranform it in some way and output something that can't be used to reverse engineer that orig string. And that if we apply this function again to the same secret string, we get back the same transformation.

Here is an example bcrypt output

$2a$10$vI8aWBnW3fID.ZQ4/zo1G.q1lRps.9cGLcZEiGDMVr5yUP1KUOYTa

Here are the components

- 2a is the version of bcrypt
- 10 is the cost factor; 2^10 iterations of the key derivation function are used
- vI8aWBnW3fID.ZQ4/zo1G.q1lRps.9cGLcZEiGDMVr5yUP1KUOYTa is the salt and the cipher text, concatenated and encoded in a modified Base-64. The first 22 characters decode to a 16-byte value for the salt. The salt is random.  The salt is there to prevent rainbow table attacks. The remaining characters are cipher text to be compared for authentication

When someone first enters their pwd:
- derive a key using the password and salt and cost.
- use the key to encrypt the string for as many iterations as the cost. Store the cost, salt, and cipher text. (Each part has a fixed length so we can split them back out later.

When someone tries to auth:
- split out the cost and salt
- use them to derive a key from the password attempt
- if the generated string from using this key matches the stored cipher text, it's a match

Resources
- https://en.m.wikipedia.org/wiki/Bcrypt
- http://stackoverflow.com/questions/6832445/how-can-bcrypt-have-built-in-salts

----------------------------------
Cookies and sessions
----------------------------------
- Would like http to be a bit less stateless
- Know who a visitor is across requests
- In sinatra, you access cookies thru the session
- Session is a hash like object available in all ur contollers. You can assign keys and values. When you assign them, sinatra will write a cookie for your domain in the users browser
- Every request the browser makes on that domain, the cookie will be sent along
- So you can set a unique value in the session and check who ur visitor is


----------------------------------
Implementation notes
----------------------------------

- Include bcrypt gem in ur Gemfile
- bundle
- set up user table in db to have password_hash field
- Override getters and setters in the user model

For our controllers and views, we need to

1. Show a form to register a user
- get /users/new

2. Handle a user registration form submission
- post /users
- create the hashed password here (thru the use of the User model)

3. Show a login form
- get /sessions/new OR get /login

4. Handle a login form submission
- post /sessions
- post /login
- session[:user_id] = @user.id

5. Handle a logout button
- delete /sessions/id=84534 OR delete /sessions OR post/delete /logout
- session[:user_id] = nil

Aaaand LASTLY

6. Access the user from any route
```
if session[:user_id]
  @user = User.find(:user_id => session[:user_id])
   # they are logged in
else
   # no one is logged in
```
- Use a helper method something like
````
def current_user
  if session[:user_id]
    return User.find(:user_id => session[:user_id])
end
```

- In some controller where i have to check the user_id
```
get '/users/:id'
  if current_user
    # render the profile
  else
    # send them to the login page
  end
end
```
