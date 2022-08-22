+++
author = "Nick"
categories = ["api", "api-resource"]
date = 2021-12-29T17:37:00Z
description = ""
draft = true
slug = "api-security-the-basics"
tags = ["api", "api-resource"]
title = "API Security - The basics - OWASP"
url = "/api-security-the-basics" 

+++


As businesses become more and more reliant on integrated software systems they rely more and more on systems with API routes. In this post I hope to cover the basics of of the OWASP Top 10 [API Security Project](https://owasp.org/www-project-api-security/).

This will be by no means a comprehensive post about the each item in the top 10 but more of a primer and introduction to those topics. I'll post a series of more in-depth articles about each of these topics.

So why are we looking at the API top 10 and not say the 'standard' [OWASP Top 10](https://owasp.org/www-project-top-ten/)? Well, at this point in the game, if you are a developer you should already be familiar with the OWASP Top 10. What I'm finding is the more I talk to companies and developers in those companies is that they have no idea that the API top 10 even exists!

Let's take a look at the top 10:

* API1:2019 Broken Object Level Authorization
* API2:2019 Broken User Authentication
* API3:2019 Excessive Data Exposure
* API4:2019 Lack of Resources & Rate Limiting
* API5:2019 Broken Function Level Authorization
* API6:2019 Mass Assignment
* API7:2019 Security Misconfiguration
* API8:2019 Injection
* API9:2019 Improper Assets Management
* API10:2019 Insufficient Logging & Monitoring

I'm willing to be that if you took a look at the above, you could probably find these flaws in your projects in one way or another.

## API1:2019 Broken Object Level Authorization

APIs tend to expose endpoints that handle object identifiers, creating a wide attack surface Level Access Control issue. Object level authorization checks should be considered in every function that accesses a data source using an input from the user.

This can get to be a very complex item. At it's core, it's referring to IDOR's (Insecure Direct Object References). This is the concept that an unauthorized person can obtain data from other users in that system.
The two most common types of Broken Object Level Authorization are those based on user ID and those based on an object ID.

A basic example of UserID being broken:
```python
# In this example we need to check the user_id from the GET parameter
# is the same as the user_id from the object's owner

if (get.request(user_id) == object.ownerID):
    RenderData()
else:
    print("Access Denied")
    os.sleep(3)
    Redirect(landingpage)
    
# the above is an OKish method for checking user ID and displying data
# the below will be a poor way of writing the method

if (get.request(not user_id) == object.ownerID):
    print("Access Denied")
    sleep(3)
    Redirect(landingpage)
    
RenderData()  # this RenderData function is OUTSIDE the if statement, just don't
```

Now in theory, this will not get to the `RenderData` function because of the `not` in the `if statement`, however - this function could get called by hitting the back button and and the browser will often call the function (based on the stack you are using).

Next we take a look at broken Object ID. This sort of vulnerability exists when Object ID's are passed to the server and the server is not checking authorization level for the object. This seems to be most common when working with legacy code. Often there is insufficient documentation of a function or when development of a function begins. As time goes on, knowledge of this function is lost  and might need to be re-written. This is when its easy to forget about authorization requirements or sometimes conflicting authorization methods are created, causing an overwrite or null authorization flow.

When one functionality integrates with another, it is easy to overlook certain security considerations. This is because the two functionalities are developed separately and often by separate teams. This makes it fairly easy to overlook proper security checks.

An example of broken object ID. We have some functionality that adds products to a store. Users should only be able to edit there own products. 

We can `GET prodcut?id=2` to get product details. 
We can `POST product?id=2` to update the product details. 
We can `DELETE product?id=2` to delete the product.

These alone are all assumed secure since users can only execute on there own objects. Later on down the line, we want to add a bulk import option:

`POST /import?file=items.csv`

The problem here is that the developers forgot to check if the user is allowed to write to that product. Leaving a potential attacker able to write to products that do not belong to them.

## API2:2019 Broken User Authentication

This is another fairly broad topic. It is not easy to implement authentication, especially if you are creating your own auth system. There are many companies that's sole purpose is to supply a proper authentication mechanism for your product. I am a big proponent of using these library's and toolsets offered out there. While none are perfect - they help tremendously in reducing the attack vector of your product.

There are quite a few best practices here for authentication endpoints and we'll go over some of them below.

Example 1 - Password Recovery Attack:

We have an endpoint for reseting a password
```
POST /api/v2/reset-password
{
    userID=69420
}
```

Now the user will receive a token or token link in their email and then supply a new password.

```
POST /api/v2/reset-password
{
    userID=69420
    token={TOKEN}
    newPassword={SUPPLIED}
}
```

Now if there is no Rate Limiting or Captcha on this endpoint, the token could be bruteforced. 

Example 2 - JWT Validation accepts 'None':

So, you probably already know this, but your JWT endpoint should **always** validate the given token with the proper algorithm.



