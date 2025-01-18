---
layout: post
title:  "Try out OAuth 2.0 auth-code flow by your own"
date:   2024-01-18 05:25:00 +0530
categories: jekyll update
published: true
---
<div style="text-align: center;">
  <img src="\images\2024-01-18-Oauth-flow-pkce\PKCE_WOW.jpg" style="width: 500px; height: auto;">
</div>
    <br>
<span>Courtesy: bing/create</span>



---

<br>
<br>
The best way to learn and understand the OAuth 2 flows, is by running them by your own with simple steps.

If you are using the sdks for your application, the app will handle all the flows and fullfil all the requirements to get the token for you.


Of course, we don't need to reinvent the weel, but sometimes it is also true that people used to talk about complexities and leave it to the machines and systems.

But we know all complex systems are built on, by the aggregation of simple steps on top of eacch other.

So, if I look at the authcode flow diagram in  OAuth 2 [website](https://www.oauth.com/oauth2-servers/pkce/), The documentation felt intimidating, but the diagram was something enticed to be adopted
```
                                                 +-------------------+
                                                 |   Authz Server    |
       +--------+                                | +---------------+ |
       |        |--(A)- Authorization Request ---->|               | |
       |        |       + t(code_verifier), t_m  | | Authorization | |
       |        |                                | |    Endpoint   | |
       |        |<-(B)---- Authorization Code -----|               | |
       |        |                                | +---------------+ |
       | Client |                                |                   |
       |        |                                | +---------------+ |
       |        |--(C)-- Access Token Request ---->|               | |
       |        |          + code_verifier       | |    Token      | |
       |        |                                | |   Endpoint    | |
       |        |<-(D)------ Access Token ---------|               | |
       +--------+                                | +---------------+ |
                                                 +-------------------+

                     Figure 2: Abstract Protocol Flow
                     (From OAuth 2community website)

```
 No further explanation, we can try some realtime calls and learn by practice

> **Prerequisites**  
>WSL in Windows to run some `curl` commands  
>Any OAuth 2 server (I preffered okta thos time you can find a free account to open [here](https://developer.okta.com/))
>A cliend id and registered user for some trial outs
>A Browser to get some redirection urls

Hands on

---

Run these bash commands to generate the `CODE CHALLENGE` and    `CODE VERIFIER`.

```
# Generate Code Verifier
CODE_VERIFIER=$(openssl rand -base64 32 | tr -d '=+/')

# Generate Code Challenge
CODE_CHALLENGE=$(echo -n $CODE_VERIFIER | openssl dgst -sha256 -binary | openssl base64 | tr -d '=+' | tr '/+' '_-')

echo "Verifier: $CODE_VERIFIER"
echo "Challenge: $CODE_CHALLENGE"```
```


Check you domain name of your Okta auth server and create a url like this:
```
https://dev-23171555.okta.com/oauth2/default/v1/authorize?
client_id=0oamjhc3duhQI46bi5d7&
response_type=code&
scope=openid%20profile%20email&
redirect_uri=http://localhost:5173/profile&state=1234&
code_challenge_method=S256&
code_challenge=stvsdDox4UGFke3kx6qlYViBRhmkpgPqJ6TsnO6AZTA

```
You will be redirected to the Oauth Server page where you can log in with your registered user and password.

Boom...
Your browser says, something broken, but if you look the url, you can see that you have been redirected to the configured url, with some parameters:

```
http://localhost:5173/profile?code=svHkgr1ksofnXmySrd0ylGwQur7keLzJI1mxfkf3y5A&state=1234
```

Yeah, you got the code and the state to verify with the state of your request.

Code is what we need to exchange for a toke.

Lets do that next.

In order to achieve it, we need to make a `POST` call to the token endpoint of our auth server.

```
curl -X POST "https://dev-23171555.okta.com/oauth2/default/v1/token" \
  -H "Content-Type: application/x-www-form-urlencoded" \
  -d "grant_type=authorization_code" \
  -d "client_id=0oamjhc3duhQI46bi5d7" \
  -d "code=svHkgr1ksofnXmySrd0ylGwQur7keLzJI1mxfkf3y5A" \
  -d "redirect_uri=http://localhost:5173/profile" \
  -d "code_verifier=$CODE_VERIFIER"
```
the client id, code and verifier are included as the `post` body parameter.

If you made the call successfully, you should get a response similar to this:

```
{
  "token_type": "Bearer",
  "expires_in": 86400,
  "access_token": "9t6P3uV95az8oTzop-FupgjdfhksTA-_RZUnablsPkX85stuNLvx5l0KLWg1vCRMLIIdSf9m",
  "scope": "photo offline_access",
  "refresh_token": "hVJlxyMdq2LyJ8aSkv0mhMiq"
}
```
You got what you need to access your resources following a PKCE flow.

That was a simple walk through demonstration of what an OAuth Server PKCE.

Hope this helps!

Happy Coding....
