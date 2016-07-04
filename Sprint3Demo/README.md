# Node JS template application for OAuth

This is a sample application where the developer can start building the application using the OAuth microservice. This application provides a sample user interface, implementation of the login and the data hooks configured in the UI. The developer can modify the UI and add the required business functionality and can push and restage the application into target PaaS platform.  The OAuth microservice expose API end points methods where the developer can login, logout, send and validate the OTP using Twilio or Sendgrid

### Template application files

File | Description
:-- | :-- 
src/app.js | This file does all the bootstrapping by require-ing all the controllers, models and middlewares. Configures the port and domain, starts the application on the available or given port.
src/config.json | Sample config file to check which providers are enabled.
src/hook.json | config file to refer for sendgrid and twilio account details.
views/ | Contains the jade view files for different scenarios.
package.json | All npm packages contain a file, this file holds various metadata relevant to the project.


### Application configuration

- Find the required credentials of the AuthService from the VCAP_SERVICES. The credentials will contain the following details

      ```
       serviceUrl: Base URL for authentication service.
       *provider*_callback: This URL is the provider callback URL. Configure this URL as redirect URI in the provider app configuration. Keys will be in the form of facebook_callback, google_callback, linkedin_callback, twitter_callback
       apiKey: This Secret key is required for consuming the authentication services APIs. Do not share this key, keep it confidential
      ```

### Application execution

- app.js provides the sample routes calling the services of Authentication service.

      ```
       1. '/login' to show the provider options to login.
       2. '/OAuth' Initiates the call to Authentication service based on a selected provider login.
       3. '/callback' Authentication service redirects back to the application with accessToken and profile information. Later it is rendered to show some profile information.
       4. '/logout' to terminate an existing login session and redirect to login page.
      ```

- After all configuration is done, start the application and try 'YOUR_DOMAIN_URL' or 'YOUR_DOMAIN_URL'/login in the browser. For eg: http://localhost:3000/login or http://localhost:3000/


## Authentication Service API

## GET /facebook, /google, /twitter, /linkedin
Authenticates the user against requested provider and redirects back to application with a code.

### Request
| Query Param  |                  Description                                                          |
|--------------|---------------------------------------------------------------------------------------|
| callbackUrl  | callbackUrl to where Authentication service needs to redirect after authentication    |

### Response
| HTTP           |      Value                           |
|----------------|--------------------------------------|
| query params   | code=eyJ0eXAiOiJKV1dfgdfg5fggyhh...  |

* Example API call

      ```
       var url = "http://authservice.54.208.194.189.xip.io"+"/facebook?"+"callbackUrl=http://localhost:3000/callback";
       res.redirect(url);
      ```

## POST /account
This API retrieves the account details of the logged in user. Request parameters of the body are code(received from /facebook, /google, /twitter, /linkedin in callback) and apiKey

### Request
| HTTP       |                             Value                                           |
|------------|-----------------------------------------------------------------------------|
| Body       | {"code":"eyJ0eXAiOiJKV1dfgdfg5fggyhh...","apiKey":"YOUR_API_KEY_FROM_VCAP"} |

### Response
| HTTP       |  Value                                                               |
|------------|----------------------------------------------------------------------|
| Body       | {"accessToken":"Rcp4xZC06HnmcFMaW866wS9KSK4bCjxBQB2UTPfVbOJH","id":"607861336036","displayName":"John Doe","provider":"facebook"}|

*Note: refreshToken will be available only for /twitter call. Not applicable for other providers


## GET /logout
Terminates an existing login session and redirects to the callback URL

### Request
| Query Param  |                  Description                                                          |
|--------------|---------------------------------------------------------------------------------------|
| callbackUrl  | callbackUrl to where Authentication service needs to redirect after authentication    |

### Response
- Redirects back to the callbackUrl after successful session termination. No query params will be passed.


## POST /generate
- This API generates an OTP with the given combination(numeric,alpha or both) and sends the OTP to desired receipient based on the given channel and receipient details.
- Supports SendGrid and Twilio as channels.

### Request
| HTTP       |                             Value                                               |
|------------|------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| Body       | {"channel": "sendgrid", "otp": {"otpLength": 5,"otpType": "numeric","otpExpiryTime":  4},"sendgrid": {"accountSID": "1234","authToken": "abcd","toRecipient":"someemail@gmail.com","fromMail": "someemail@gmail.com"}} |

### Response
| HTTP       |  Value                                                             |
|------------|--------------------------------------------------------------------------------------------|
| Body       | {"otpCode":"79653","otpKey":"69687d70cb2503339e780f54db7a02bb958e86f84aeaa6023dc7397c531"} |

* Example JSON body for request

      ```
       1. for sendgrid : {"channel": "sendgrid", "otp": {"otpLength": 5,"otpType": "numeric","otpExpiryTime":  4},"sendgrid": {"accountSID": "1234","authToken": "abcd","toRecipient":"someemail@gmail.com","fromMail": "someemail@gmail.com"}}
       2. for twilio : {"channel": "twilio", "otp": {"otpLength": 5,"otpType": "numeric","otpExpiryTime":  4},"twilio":{"accountSID": "hdfk6545f47d11a6a7a56","authToken": "3505f762yujy655c727fe","toRecipient": "somenumber","fromNo": "registerd_number"}}
      ```
* Note : Default values for otpLength, otpType and otpExpiryTime are 4, numeric and 10 respectively. otpType can be of numeric or alpha or alphanumeric

## POST /validate
- This API validates the requested OTP. Response of /generate API is the body request for this API

### Request
| HTTP       |                             Value                                                          |
|------------|--------------------------------------------------------------------------------------------|
| Body       | {"otpCode":"79653","otpKey":"69687d70cb2503339e780f54db7a02bb958e86f84aeaa6023dc7397c531"} |

### Response
| HTTP       |  Value                                      |
|------------|---------------------------------------------|
| Body       | {"status":"OTP is validated successfully"}  |


## POST /saveLog
- This API saves logs to Graylog server or MongoDB based on the user choice.

### Request
| HTTP       |                             Value                                                          |
|------------|--------------------------------------------------------------------------------------------|
| Body       | {"level":"INFO","message":"Info message", "appid":"OneC"} |

### Response
| HTTP       |  Value                                      |
|------------|---------------------------------------------|
| Body       | {"status":"success", "message":"Successfully sent to Graylog server"}|


## Sample UI for testing pre-hook and post-hook
- Pre Hook

      ```
       1. By default index.jade is used for login. Render to index_With_PreHook.jade from /login to show some sample UI to generate OTP as pre hook. Modify src/hook.json key "prehooktemplate" as "index_With_PreHook"
       2. Modify ajax call URL in index_With_PreHook.jade to use either sendgrid or twilio. /generateOTPWithTwilio to use twilio and /generateOTPWithSendGrid to use sendgrid (url: '/generateOTPWithTwilio' or url: '/generateOTPWithTwilio')
       3. Modify src/hook.json for twilio and sendgrid details
      ```

- Post Hook

      ```
       1. By default account.jade is used to show some account information. Render to account_With_PostHook.jade from /callback to show some sample UI to generate OTP as post hook.  Modify src/hook.json key "posthooktemplate" as "account_With_PostHook"
       2. Modify ajax call URL in account_With_PostHook.jade to use either sendgrid or twilio. /generateOTPWithTwilio to use twilio and /generateOTPWithSendGrid to use sendgrid (url: '/generateOTPWithTwilio' or url: '/generateOTPWithTwilio')
       3. Modify src/hook.json for twilio and sendgrid details
      ```
