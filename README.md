# Play Share App
# 📋 APP OVERVIEW:

* This is a Reddit/Imgur-like app where gamers can share short clips of their gameplays. 
* REST API server is built with Node, Express, MongoDB, Redis. Will migrate database to PostgreSQL in the future. Client is currently being built with React.
* Hosted a clustered REST API server on DigitalOcean and used NGINX as a reverse Proxy. Enabled HTTPS. https://playshare.cloud/ (currently disabled) 

# 💎 FEATURES:
  * **Rate Limiting** to protect against basic DDOS attacks: blocking an IP for several minutes if they make too many requests. 
  * **Session Persistence:** Keeping users logged in by silently refreshing tokens
  * **Client-side Protection:** Access and refresh tokens tokens are stored in HttpOnly cookies to prevent client from accessing them. 
  * **TLS Handshake:** Optional TLS handshake implementation (implemented in server, deleted implementation on client) 
  * **Some Others:** Caching data for frequent endpoints, Input validation, Password hashing, etc

# DEMO:

![App security demo (unfinished)](/PicturesGifs/App_demo_unfinished.gif)
<p align="center" style="font-style: italic">
    Fig 1: App security demo so far (App still in development)
</p>

* **What users can do:** <details>      
  <summary > Click to expand section  </summary>   

  * **Users** can make a posts, edit their own posts, delete a post, see all of their posts, and like other user's posts. User feed is currently in production. Uploading video and images to S3 bucket in development. 
  * **Admin** can see all user's posts, see only a specific user's posts, and delete one or many posts by id. 
  * **(In Progress)** Users can upload image/video to Amazon S3 bucket. Users can delete their own post, can upvote/downvote other posts, comment on other user's posts. 
  * **(Future Plans)** Users can join different game groups just like reddit and follow users. App will feature an hierarchical commenting system and messaging system in the future.
 
  </details>    

<br/>

# 📌 TECHNOLOGIES / DEPENDENCIES (REST API):
* The REST API Server is built using **Node**, **Express**, and **Mongoose**
* The Client side is still in production and is being built with **React**
* **Redis (async-redis)** - used to cache requests, decrease response times. 
* **helmet.js** - used to give some basic security to REST API application.
* **express-rate-limit** - used to limit how many requests can be made to the server in a specified time by the same IP
* **JWT** - used to authenticate a user - used to make user access and refresh tokens.
* **Joi** - used to validate request body.
* **cookie-parser** - used to parse refresh token signed cookie data from request.
* **node-rsa** - used to create asymmetric RSA keys to initiate TLS handshake between client and server. 
* **bcrypt** - used to store hashed passwords into the database.
* **crypto-js** - (not used if using HTTPS and disabling TLS) used to encrypt response and decrypt request using the client's symmetric key (AES).
* **morgan** - used to log endpoint response times to optimize code. 

# 🚑 SERVER RESPONSE CODES:
  ![Silent Refresh](/PicturesGifs/Basic_Response.PNG)
  <p align="center" style="font-style: italic">
    Fig 2: Responses of the API Server: 1, -1, -2, 2, -3
  </p>

# 🏠 RUN SERVER LOCALLY:
1) Rename ***.env.example*** to ***.env***. Can modify all eight variables but must change the **DB_CONNECT** variable so that you can connect to your Mongo Database. Make sure the keys are long and randomly generated. 
    <details>      
      <summary> Description of the variables</summary>
    
      * `DB_CONNECT`  - Store your MongoDB Connection
      * `ADMIN_USERNAME` - Email address of the admin account.
      * `ADMIN_SECRET_KEY` - This will be used to make the admin's access JWT
      * `USER_SECRET_KEY`  - This will be used to make the admin's and user's access JWT
      * `REFRESH_TOKEN_SECRET` - This is used to generate a refresh JWT refresh
      * `COOKIE_SECRET` - This is used to sign HttpOnly cookies
      * `SALT_NUM = 10` - Can keep this as is. This is the salt number to hash the password and the JWT User Secret Key to store in the database. Can change this number every year to change 
      the hashing algorithm of these fields.
      * `USE_TLS = false` - Can keep this as is. Do you want to use the TLS handshake? false = disable TLS (do this when using https). true = enable TLS. 
    </details>
2) `redis-server` - download redis and start the redis server for REST API.
3) `npm install` on the **CLIENT** & **SERVER** directories
4) `npm start` on the **CLIENT** & **SERVER** directories to run the client and server 
<br/>

# 🛡️ APP SECURITY:
<details>      
  <summary> APP SECURITY SUMMARY </summary>

  * **Rate Limiting** Limited requests to 100 requests every 10 minutes. This will guard against simple DDOS attacks by rating how many requests can be made in a specific time by the same IP.
  * **Input Validation** with **Joi**.
  * **Passwords hashed in Database:**
  * **Long Secret Keys:** The secret keys needed to make tokens, cookies, and hash passwords are 700-1200 characters long and are stored in the **.env** file. The keys are created using concatenations of several randomly generated hashes. 
  * **Session Persistence:** Application can keep users logged in if the client supplies the correct refresh token HttpOnly cookie and the correct `username` header. 
  * **Cors & Helmet Protections:** **Cors** and **helmet.js** middlewares provide some basic security to server.
  * **HttpOnly Cookies:** Token cookies are HttpOnly cookies with flags set to `httpOnly=true`, `secure=true` to ensure the client cannot read its contents. 
  * **Token Expire Times:** Access token expires 5 minutes and cookie expires in 1 day. Refresh token and cookie expires in 15 days. 
  * **TLS:** (optional if using TLS) All data in requests and responses are AES encrypted by the symmetric key. Api automatically decrypted request with symmetric key.
  </details>

  ## 🍪 A) Authentication via JWT Access & Refresh Tokens + Silent Refresh to Persist Sessions:
  ![Silent Refresh](/PicturesGifs/Silent_Refresh.png)
  <p align="center" style="font-style: italic">
    Fig 3: Silent Refresh Process to persist user session. Server will refresh access and refresh tokens if client pass all requirements. 
  </p>

  * After successful login, access token and refresh tokens are made and stored in a signed HttpOnly cookie so that they cannot be accessed in the client. Can silently refresh tokens to persist sessions with: (1) valid refresh token, and (2) corresponding username in the `username` header of the request. 
  * 🍪 **Token & Cookie Creation + Expiration Times:** <details>      
    <summary > Click to expand section  </summary>

    * **Secret Keys:** 
      * **Access token** is signed with the `USER_SECRET_KEY` key if its a user or the `ADMIN_SECRET_KEY` key if it is an admin. 
      * **Refresh token** is signed with the `REFRESH_TOKEN_SECRET` key.
      * **HttpOnly cookies:** Access and Refresh tokens are stored in individual HttpOnly cookies. The cookies are signed with `COOKIE_SECRET`.
    * **Token Payload:** The payload of the tokens is the username along with a randomly generated number. Example: `{username: 'Tom', id: '1234'}`
    * **Expiration Times:** 
      * **Access token** expiration time: 5 minutes
      * **Refresh tokens** expiration time: 15 days
      * **Access tokens HttpOnly cookie** expiration time: 1 day 
      * **Refresh token HttpOnly cookie** expiration time: 15 days
    </details> 
   
  * 🤫 **Silent Refresh Procedure to Persist Sessions (Figure 3)**: <details>      
    <summary > Click to expand section  </summary>

    1) When a request has an invalid access token, the server will verify if the refresh token is valid. If it is valid, the server will respond with status code `-2`. 
    2) Client will send a GET request to the `/auth/refresh` endpoint. 
    3) Server will decrypt the refresh token and will get the `username` and `id` fields from the payload. It will fetch the value of `username-id` from the database. If the incoming refresh token matches the token saved in the database, and the `username` header matches the `username` payload field of the token, the server will try to refresh the tokens.
    4) If the server successfully refreshed the tokens, it will respond with status code `2` and will delete the token from the database and will add the new token to the database. If unsuccessful, server will respond with `-1`.
    </details>  

  * **Authentication & Authorization** <details>      
    <summary > Click to expand section  </summary>

    * Multiple checks to authenticate user: 
    1) Validating access and refresh tokens. 
    2) Matching token payloads with username header to ensure that the correct user is using the token. 
    3) Checking if user is in the database
    4) Checking if refresh token is in database (used request is trying to access admin routes, when refreshign tokens, or when access token is invalid)
  </details>

  

## 🤝 B) TLS handshake (optional, disabled by default since using https):
  ![TLS Handshake](/PicturesGifs/TLS_Handshake2.png)
  <p align="center" style="font-style: italic">
    Fig 4: TLS Handshake I implemented on the server. Client in development.
  </p>

  * TLS handshake can be performed but is not needed since server and client will communicate over https. Implemented basic version of TLS for fun
    <details>      
      <summary> TLS Handshake Process </summary>

    1. Client sends initial request to server (/auth/ routes only).
    2. Server generates RSA public and private keys and send to public key to client:
      * 1) header `handshake` = 0
      * 2) header `pub_key` = public key
    3. Client generates a random hash (`SYMMETRIC_KEY`) and encrypts with public key and sends request to server with two headers: 
      * 1) header `handshake` = 0
      * 2) header `key` = `SYMMETRIC_KEY` encrypted with public key
    4. Server will then decrypt the `SYMMETRIC_KEY` with the private key and will send a response with header `handshake` = 1, signifying handshake completed for server.
    5. Client will finish by sending a request with header `handshake` = 1, signifying it has received the server's message
    6. Server will only fulfill requests for auth routes if the `handshake` header is set to 1. This means that server has the client's `SYMMETRIC_KEY` and can decrypt request. If server cannot decrypt request, the `SYMMETRIC_KEY` is incorrect and server will refuse request. 
    7. Symmetric keys are stored in a dictionary in the server (will move it to a key-value database). If user logs out, entry is deleted

    </details>


# 📐 USABILITY (CLIENT REQUESTS):
* **Client Headers** Send encrypted authentication code to server through the header <details>      
    <summary > Click to expand section  </summary>

  * To make any requests to the server, the application needs to have the valid access key.
  * `Content-Type` = `application/json`
  * `username` = username. this username will be compared to the username in the tokens to authenticate user
  * `post-id` = client will specify the post id here to edit the specific post
  * `like-dislike` = `dislike` or `like`
  * (optional if using TLS) `handshake` =  
    * nothing - to initiate TLS handshake
    * `0` - to  say sending client's symmetric key to server 
    * `handshake_index` - this is sent by server after successful TLS handshake 
  * (optional if using TLS) AES encrypt the post request body with the symmetric key

  </details>  


  
  


