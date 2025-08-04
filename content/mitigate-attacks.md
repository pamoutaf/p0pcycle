---
title: "Mitigate attacks on your web application"
draft: false
tags:
  - hacking
  - web application
  - security
date: 2024-03-17
---

Depending on the technologies used on your web application, you could be vulnerable to multiple different types of injections. We will discuss 4 popular injection attacks and how to mitigate them. Most of the information from this blog comes directly from OWASP and PortSwigger.

Small confession… I wrote this blogpost mostly for myself, to study my material and better understand these concepts.

Maybe this blogpost will motivate you to learn and try these injections. As referred to in my resources page, you can visit the PortSwigger Academy to play around with different injection attacks on vulnerable web applications.

## 1. XXE (xml external entity)
**What is it**

XML is a language used for storing and moving data within your server. It’s not as popular as JSON, but some applications still use it to structure data when it’s sent from one place to another, typically over the internet. In XML, there’s something called a Document Type Definition (DTD). This is a set of rules that helps define what kind of data can be in an XML document, and what it means. It’s like a blueprint for understanding the data. You can use DTD to declare custom XML external entities that the XML parser will interpret.

```xml
<!DOCTYPE vuln [ <!ENTITY xxe SYSTEM "file:///etc/passwd"> ]>
```
‘xxe’ is defined as a custom external entity.

An XXE injection is a vulnerability that allows an attacker to mess with the processing of data handled by the application. Generally, you can extract data from the server’s filesystem, and communicate with other systems that the application uses. An XXE vulnerability can also be leveraged to perform further attacks such as Server-side request forgery (SSRF).

**Mitigations**

You can disable dangerous XML features that are supported by the application’s XML parsing library. Features include but are not limited to:

 - External Entity Declaration;
 - DTD Parsing;
 - XInclude.
 - PortSwigger and OWASP say it is generally enough to just disable support for XInclude and DTD’s to protect against XXE attacks.

OWASP has a Detailed XXE Prevention guide for multiple programming languages.

## 2. XSS (cross site scripting)
**What is it**

Cross-site scripting (XSS) is a vulnerability which could allow an attacker to compromise user interactions with the application. Generally, JavaScript is used to execute malicious code inside of a victim’s browser and compromise their interaction with the application.

```javascript
<script>alert("xss")</script>
```
There are 3 common XSS attacks, which I will mention but not go into detail of how they differ as this is not the point of this blogpost. They can be categorized as such:

 - Reflective;
 - Stored;
 - DOM-based.

**Mitigations**

You can prevent XSS attacks by encoding data on the output or by validating the input data before it is processed by the application.

On input validation, it’s better to use whitelisting instead of blacklisting. Whitelisting means you only allow specific actions that you know are safe, while blacklisting means you try to block everything that seems dangerous. With whitelisting, it’s easier to keep track of what’s allowed, and you won’t accidentally forget to block a risky parameter. Plus, it keeps your defenses strong even if new vulnerable protocols arise.

You can also prevent XSS attacks using the Content Security Policy (CSP) header. This will stop scripts from outside of the webpage from loading and decide if scripts inside the webpage can run. Where possible, try to host resources on your own domain. CSP can also be used to protect against clickjacking attacks. Contrary to the X-Frame-Options header which is generally used for preventing clickjacking, the CSP header can specify multiple domains and use wildcards using the “frame-ancestors” directive.

```http
Content-Security-Policy: default-src 'self'; img-src 'self' cdn.example.com; frame-ancestors 'self' https://normal-website.com https://*.robust-website.com;
```
> The Content-Security-Policy header allows you to restrict which resources (such as JavaScript, CSS, Images, etc.) can be loaded, and the URLs that they can be loaded from.

[CSP polic](https://content-security-policy.com/)
OWASP has a guide on how to configure your CSP header to prevent XSS attacks.

## 3. Structured Query Language (SQL)
**What is it**

Structured Query Language is used to manage data in databases. The SQL injection attack could allow attackers to mess with the queries that an application makes to its database. It could be used to access data stored in the application’s database that is normally not visible to regular users, such as passwords or personal user information.

```sql
SELECT name, description FROM products WHERE category = 'Gifts' UNION SELECT username, password FROM users--
```
**Mitigations**

Most SQL injections can be prevented using parameterized queries, which blocks the user input from interfering with the query structure. This method can be used where input is considered as data within the query.

A parameterized query is basically allowing a place holder for user input to be added to the query. Instead of concatenating user input into the query directly (= bad), you allow a placeholder for user input to be added to the query.

```sql 
PreparedStatement statement = connection.prepareStatement("SELECT * FROM products WHERE category = ?");
statement.setString(1, input);
ResultSet resultSet = statement.executeQuery();
```
On the other hand, using parameterized queries cannot stop untrusted input in tables, column names, or the ORDER BY clause.

```sql
SELECT * FROM ? WHERE id = ?
```
In this scenario, if we try to use a parameterized query for the table name, it won’t work because table names can’t be parameterized. To mitigate this, you can use a whitelist to specify allowed input values.

Another way to improve security is to follow the Least Privilege principle. This means giving database users and applications only the permissions they absolutely need, and not giving them more privileges than necessary. This helps to reduce the chances of unauthorized access or misuse of data.

OWASP has a guide on how to implement protections against SQL injections.

## 4. Server-side request forgery (SSRF)
**What is it**

Server-side request forgery is a vulnerability that allows an attacker to cause the application’s server to make requests to another location. Attackers often attempt to establish connections to internal services within an organization’s infrastructure, which are typically protected from external access by a firewall.

Think of the web application as a bridge connecting the organization’s internal systems. In an SSRF attack, the attacker exploits this bridge to gain access to sensitive information stored within an organization’s internal systems through the web application hosted on those internal systems.

```sql
POST /product/stock HTTP/1.0
Content-Type: application/x-www-form-urlencoded
Content-Length: 118
```

```html
stockApi=http://stock.weliketoshop.net:8080/product/stock/check%3FproductId%3D6%26storeId%3D1
```
In this example, we are asking the server to make a request to the URL in order to retrieve the stock status of a product. An attacker could manipulate the URL and replace it with:

```html
stockApi=http://localhost/admin
```
If you were to directly request this page to the server, it would probably return an unauthorized error back to the user. But in the SSRF scenario, the application is receiving a request from the local machine to retrieve information from the administrator’s page. This technique bypasses access controls since the requests seems to come from a trusted source (internal system).

**Mitigations**

Within the network, it’s recommended to segment resources in different areas to make it harder for hackers to access sensitive data and use a firewall to block any suspicious or unnecessary traffic.

From the web application, input validations can be added to ensure that the user input follows the business/technical format which is expected.

OWASP has a [guide](https://cheatsheetseries.owasp.org/cheatsheets/Server_Side_Request_Forgery_Prevention_Cheat_Sheet.html#available-protections) on how to implement proper protection against SSRF attacks.

 
