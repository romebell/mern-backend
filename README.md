# MERN Authentication `Backend`

This is a code along for MERN Auth

Notes:
- [ ] Set up server
- [ ] Test home route
- [ ] Set up models folder and `index.js`
- [ ] Add a `.env` with your `MONGO_URI`
```text
MONGO_URI=mongodb://localhost:27017/myApp
```
- [ ] Set up `User` models
- [ ] Make controllers folder and test /test route
- [ ] Setup passport strategy
- [ ] Intialize passport and pass passport as arguemnt to config
- [ ] Make controllers and routes for user
   - [ ] /test, /register, /login, /profile
   - [ ] Test each one after completing it in Postman

## What it includes

* Mongoose User schema and model
* Settings for the database
* Passport and passport-jwt for authentication
* JSON Web Token
* Passwords that are hashed with BCrypt

### User Model

| Column Name | Data Type | Notes |
| --------------- | ------------- | ------------------------------ |
| _id | Integer | Serial Primary Key, Auto-generated |
| name | String | Must be provided |
| email | String | Must be unique / used for login |
| password | String | Stored as a hash |
| timesLoggedIn | Number | used to track the amount of times a user logs in |
| date | Date | Auto-generated |
| __v | Number | Auto-generated |

### Default Routes

| Method | Path | Location | Purpose |
| ------ | ---------------- | -------------- | ------------------- |
| GET | / | app.js | Server file |
| GET | /users/test | users.js | Return json data |
| POST | /users/login | users.js | Login data |
| POST | /users/signup | users.js | Signup data |
| GET | /users/profile | users.js | Profile data |
| GET | /users/all-users | users.js | Get all users |

### Alternate `signup` Controller
This alternate `controller` is returning back a token to allow the user to interact with the app immediately with a token instead of having the user login right after signing up.

```js
const signup = async (req, res) => {
    console.log('--- INSIDE OF SIGNUP ---');
    console.log('req.body =>', req.body);
    const { name, email, password } = req.body;

    try {
        // see if a user exist in the database by email
        const user = await User.findOne({ email });

        // if a user exist return 400 error and message
        if (user) {
            return res.status(400).json({ message: 'Email already exists' });
        } else {
            console.log('Create new user');
            let saltRounds = 12;
            let salt = await bcrypt.genSalt(saltRounds);
            let hash = await bcrypt.hash(password, salt);

            const newUser = new User({
                name,
                email,
                password: hash
            });

            const savedNewUser = await newUser.save();

            const payload = {
                id: savedNewUser.id,
                email: savedNewUser.email,
                name: savedNewUser.name,
            }

            // token is generated
            let token = await jwt.sign(payload, JWT_SECRET, { expiresIn: 3600 });
            let legit = await jwt.verify(token, JWT_SECRET, { expiresIn: 60 });

            res.json({
                success: true,
                token: `Bearer ${token}`,
                userData: legit
            });

        }
    } catch (error) {
        console.log('Error inside of /api/users/signup');
        console.log(error);
        return res.status(400).json({ message: 'Error occurred. Please try again...'});
    }
}
```

## `users.js`
```js
require('dotenv').config();
const express = require('express');
const router = express.Router();
const bcrypt = require('bcrypt');
const jwt = require('jsonwebtoken');
const passport = require('passport');
const JWT_SECRET = process.env.JWT_SECRET;
```
