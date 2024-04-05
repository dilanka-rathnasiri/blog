# Introduction to Authentication and Authorization

> cover image: Photo by <a href="https://unsplash.com/@introspectivedsgn?utm_content=creditCopyText&utm_medium=referral&utm_source=unsplash">Erik Mclean</a> on <a href="https://unsplash.com/photos/selective-focus-photography-of-gray-metal-padlock-XUJcmgEhpjA?utm_content=creditCopyText&utm_medium=referral&utm_source=unsplash">Unsplash</a>


We will talk about the basics of authentication and authorization in this blog. I hope to write a blog series on IAM (Identity and Access Management). Understanding authentication and authorization is essential for IAM. So, this is the first article of my IAM blog series.

## 1. What is Authentication?
Authentication is the process of verifying the identity of someone or something. Simply authentication means proving identity.

Let's consider traveling from Sri Lanka to Germany by airplane. We use passports to prove our identity at the airport before leaving Sri Lanka and after arriving in Germany. This process of proving the identity using a passport is an example of authentication.

## 2. Authorization
Authorization is the process of verifying permission of someone or something to take some action. Simply that means checking whether someone can access a resource or not.

Let's go to our airplane traveling example again. Even though we have a passport, we cannot leave or enter a country. We should have legal permission to leave or enter the country. As an example, some countries have banned the passports of certain nations. So, those passport holders cannot enter the country. Our permission for leaving or entering a country is checked at the airport. This process of verifying having legal permissions is an example of authorization.

## 3. Authentication vs Authorization
Many people have a confusion about authentication and authorization. Even though authentication and authorization are related to each other in many use cases, they are different terms with different meanings.

<table>
<tr>
<th>Authentication</th>
<th>Authorization</th>
</tr>

<tr>
<td>Verifying the identity of the user</td>
<td>Checking whether the user has permission to access a resource</td>
</tr>

<tr>
<td>Password, PIN, biometrics, and authentication apps are a few ways for authentication</td>
<td>Access policy and security settings are a few ways for authorization</td>
</tr>

<tr>
<td>Usually done before authorization</td>
<td>Usually done after authentication</td>
</tr>

<tr>
<td>OpenID Connect can be used for authentication</td>
<td>OAuth 2.0 can be used for authorization</td>
</tr>

<tr>
<td>Usually, transmit information through ID tokens</td>
<td>Usually, transmit information through Access token</td>
</tr>
</table>

## Summary
We talked about the basics of authentication and authorization in this blog. These basics will help to understand the next blogs of this IAM blog series.

## Reference
[1] https://auth0.com/docs/get-started/identity-fundamentals/authentication-and-authorization
[2] https://www.okta.com/identity-101/authentication-vs-authorization/
