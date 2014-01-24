# Auth0 and Java

[Auth0](https://www.auth0.com) is a cloud service that provides a turn-key solution for authentication, authorization and Single Sign On. 

You can use  [Auth0](https://www.auth0.com) to add username/password authentication, support for enterprise identity like Active Directory or SAML and also for social identities like Google, Facebook or Salesforce among others to your web, API and mobile native apps. 

## Servlet-based Application Tutorial

This guide will walk you through adding authentication to an existing Java Web Application that is based on [Java Servlet Technology](http://www.oracle.com/technetwork/java/index-jsp-135475.html). 

In case you are starting from scratch, you can create a Java Web Application by using the `maven-archetype-webapp`

```
mvn archetype:generate -DgroupId=com.acme \
                       -DartifactId=my-webapp \
                       -Dversion=1.0-SNAPSHOT \
                       -DarchetypeArtifactId=maven-archetype-webapp \
                       -DinteractiveMode=false

```

### Maven Dependency

Let's start by adding the Auth0-servlet artifact to the project `pom.xml` file.

```xml
<dependency>
  <groupId>com.auth0</groupId>
  <artifactId>auth0-servlet</artifactId>
  <version>1.0-SNAPSHOT</version>
</dependency>
```

> Note: if you are not using Maven you can download the auth0-servlet.jar from the downloads section.

### Authentication

#### Filtering Requests

First, let's start configuring the `web.xml` found in your Web Application:

```xml
<web-app>
  ...
  <!-- Servlets -->
    ...
    <servlet>
        <servlet-name>Hello</servlet-name>
        <servlet-class>com.auth0.example.HelloServlet</servlet-class>
    </servlet>
    <servlet>
        <servlet-name>RedirectCallback</servlet-name>
        <servlet-class>com.auth0.Auth0ServletCallback</servlet-class>
        <init-param>
            <param-name>auth0.redirect_on_success</param-name>
            <param-value>/hello</param-value>
        </init-param>
        <init-param>
            <param-name>auth0.redirect_on_fail</param-name>
            <param-value>/login</param-value>
        </init-param>
    </servlet>
    <servlet>
        <servlet-name>Login</servlet-name>
        <jsp-file>/login.jsp</jsp-file>
    </servlet>

    <!-- Servlet Mappings -->
    ...
    <servlet-mapping>
        <servlet-name>Hello</servlet-name>
        <url-pattern>/hello/*</url-pattern>
    </servlet-mapping>
    <servlet-mapping>
        <servlet-name>Login</servlet-name>
        <url-pattern>/login</url-pattern>
    </servlet-mapping>
    <servlet-mapping>
        <servlet-name>Login</servlet-name>
        <url-pattern>/</url-pattern>
    </servlet-mapping>
    <servlet-mapping>
        <servlet-name>RedirectCallback</servlet-name>
        <url-pattern>/oauth2callback</url-pattern>
    </servlet-mapping>

    <!-- Filters -->
    ...
    <filter>
        <filter-name>AuthFilter</filter-name>
        <filter-class>com.auth0.Auth0Filter</filter-class>
        <init-param>
            <param-name>onFailRedirectTo</param-name>
            <param-value>/login</param-value>
        </init-param>
    </filter>
    <filter-mapping>
        <filter-name>AuthFilter</filter-name>
        <url-pattern>/hello/*</url-pattern>
    </filter-mapping>

    <!-- Auth0 Configuration -->
    ...
    <context-param>
        <param-name>auth0.client_id</param-name>
        <param-value>YOUR_CLIENT_ID</param-value>
    </context-param>

    <context-param>
        <param-name>auth0.client_secret</param-name>
        <param-value>YOUR_CLIENT_SECRET</param-value>
    </context-param>

    <context-param>
        <param-name>auth0.domain</param-name>
        <param-value>your-domain.auth0.com</param-value>
    </context-param>

    <context-param>
        <param-name>auth0.redirect_uri</param-name>
        <param-value>http://localhost:8080/oauth2callback</param-value>
    </context-param>

</web-app>
```

In the `Auth0ServletCallback` the data to popuplate principal will be persisted in session. As we will see later this can be customized.

As configured previously, the user will be redirected to `/hello`. User-provided `HelloServlet`, which overrides `doGet` method, will be handling that case.

### Login Page

Next step is to add a Login Page, let's add [Auth0 Widget](https://docs.auth0.com/login-widget2) to a jsp page called `login.jsp`.

```jsp
<!DOCTYPE html>
<html>
  <head>
    <title>Login</title>
    <script src="https://d19p4zemcycm7a.cloudfront.net/w2/auth0-widget-2.4.0.min.js"></script>
  </head>
  <body>
    <script type="text/javascript">
      var widget = new Auth0Widget({
        domain:         '<%= application.getInitParameter("auth0.domain") %>',
        clientID:       '<%= application.getInitParameter("auth0.client_id") %>',
        callbackURL:    '<%= application.getInitParameter("auth0.redirect_uri") %>'
      });

    </script>
    <% if ( request.getParameter("error") != null ) { %>
        <span style="color: red;"><c:out value="${param.error}" /> </span>
    <% } %>
    <button onclick="widget.signin()">Login</button>
  </body>
</html>
```

Point your browser to `/login` and you will be seeing that login page.

### Extensibility points

On the first part, we explained how to get running up and fast with Auth0 in your app. But, probably, you needed some degree of customization over any of the involved parts. We will see how to customize it to better suit your needs.

In order to handle the callback call from Auth0, you will need to have a Servlet that handles the request.

#### Auth0 Filter

`Auth0 Filter` can be subclassed and the following `protected` methods meant to be overriden:
 * onSuccess: What should be done when the user is authenticated.
 * onReject: What should be done when the user is not authenticated.
 * loadTokens: You should override this method to provide a custom way of restoring both id and access tokens. By default, they are stored in the Session object but, for instance, they can be persisted in databases.

Method signatures are as follows: 

```java
protected void onSuccess(ServletRequest req, ServletResponse resp, FilterChain next, Tokens tokens) throws IOException, ServletException {

}

protected void onReject(ServletRequest req, ServletResponse response, FilterChain next) throws IOException, ServletException {
        
}

protected Tokens loadTokens(ServletRequest req, ServletResponse resp) {

}
```

#### Auth0Principal

`Auth0Principal` class can be subclassed in order to expose custom information you want your principal to have.

#### Auth0ServletCallback

`Auth0ServletCallback` methods `saveTokens` and `onSuccess` can be both overriden. One, to provide the way to store accessTokens. The other, to override the onSuccess behaviour (when user is authenticated).

Method signatures are:

```java
    protected void saveTokens(HttpServletRequest req, HttpServletResponse resp, Tokens tokens) throws ServletException, IOException {
    }

    protected void onSuccess(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException {
    }
```


---

## What is Auth0?

Auth0 helps you to:

* Add authentication with [multiple authentication sources](https://docs.auth0.com/identityproviders), either social like **Google, Facebook, Microsoft Account, LinkedIn, GitHub, Twitter, Box, Salesforce, amont others**, or enterprise identity systems like **Windows Azure AD, Google Apps, Active Directory, ADFS or any SAML Identity Provider**. 
* Add authentication through more traditional **[username/password databases](https://docs.auth0.com/mysql-connection-tutorial)**.
* Add support for **[linking different user accounts](https://docs.auth0.com/link-accounts)** with the same user.
* Support for generating signed [Json Web Tokens](https://docs.auth0.com/jwt) to call your APIs and **flow the user identity** securely.
* Analytics of how, when and where users are logging in.
* Pull data from other sources and add it to the user profile, through [JavaScript rules](https://docs.auth0.com/rules).

## Create a free account in Auth0

1. Go to [Auth0](http://developers.auth0.com) and click Sign Up.
2. Use Google, GitHub or Microsoft Account to login.

## License

The MIT License (MIT)

Copyright (c) 2013 AUTH10 LLC

Permission is hereby granted, free of charge, to any person obtaining a copy
of this software and associated documentation files (the "Software"), to deal
in the Software without restriction, including without limitation the rights
to use, copy, modify, merge, publish, distribute, sublicense, and/or sell
copies of the Software, and to permit persons to whom the Software is
furnished to do so, subject to the following conditions:

The above copyright notice and this permission notice shall be included in
all copies or substantial portions of the Software.

THE SOFTWARE IS PROVIDED "AS IS", WITHOUT WARRANTY OF ANY KIND, EXPRESS OR
IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF MERCHANTABILITY,
FITNESS FOR A PARTICULAR PURPOSE AND NONINFRINGEMENT. IN NO EVENT SHALL THE
AUTHORS OR COPYRIGHT HOLDERS BE LIABLE FOR ANY CLAIM, DAMAGES OR OTHER
LIABILITY, WHETHER IN AN ACTION OF CONTRACT, TORT OR OTHERWISE, ARISING FROM,
OUT OF OR IN CONNECTION WITH THE SOFTWARE OR THE USE OR OTHER DEALINGS IN
THE SOFTWARE.
