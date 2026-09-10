---
title: "LeetCode Archiver (3): Authentication"
date: 2019-01-11T13:15:17+11:00
draft: false
categories: ["Python"]
description: "A translated technical note on LeetCode Archiver (3): Authentication, preserving the examples and context of the original article."
---
# LeetCode Archiver (3): Authentication

> Originally published in Chinese on 2019-01-11; this English edition preserves the original scope and technical context.

## Cookies and Sessions

In order to obtain our own submission records, we must first log in. But we all know that HTTP is a stateless protocol, and each of its requests is independent. Whether it is a GET or POST request, it contains all the information for processing the current request, but it does not involve changes in status. Therefore, in order to maintain a persistent state on the stateless HTTP protocol, the concepts of Cookie and Session are introduced. Both are encrypted data stored in memory or hard disk to identify user-related information.

Cookies are maintained by the client browser. The client browser will save the cookie in memory when needed. When it sends a request to the website related to the domain name again, the browser will send the URL and Cookie together as part of the request to the server. The server confirms the user's status by parsing the cookie and makes corresponding modifications to the cookie content. Generally speaking, if the expiration time is not set, non-persistent cookies will be stored in memory and deleted after the browser is closed.

Session is maintained by the server. When the client sends a request to the server for the first time, the server will create a Session for the client. When the client accesses the server again, the server will obtain relevant information based on the Session. Generally speaking, the server will set an expiration time for the Seesion. When the time since receiving the last request sent by the client exceeds this expiration time, the server will actively delete the Session.

Both methods can be used to maintain login status. For simplicity, this project currently uses Session as the method to maintain login status.

## Get data

### Analysis

First, we enter the [Login Page](https://leetcode.com/accounts/login/), open the developer tools, and check Preserve log. In order to know what data the browser submitted to the server when logging in, we can first enter an incorrect username and password to facilitate packet capture.

![7](https://raw.githubusercontent.com/chr1sc2y/warehouse-deprecated/refs/heads/main/resources/Use-Scrapy-to-Crawl-LeetCode/7.png)

By analyzing the "login/" request, we can know some key information we need, such as user-agent and referer in headers, csrfmiddlewaretoken, login and password in form data. Obviously, we can copy user-agent and referer directly, and login and password are the username and password we filled in. There is also a very unfamiliar csrfmiddlewaretoken. This is the middleware token of CSRF. CSRF is Cross-Site Request Forgery. For related knowledge, you can check [Cross-site request forgery Wikipedia](https://zh.wikipedia.org/wiki/%E8%B7%A8%E7%AB%99%E8%AF%B7%E6%B1%82%E4%BC%AA%E9%80%A0). So now we have to analyze where this token comes from.

#### Get csrfmiddlewaretoken

We copy the csrfmiddlewaretoken we just obtained and use the search function in the developer tools. We can find that this csrfmiddlewaretoken appears in the responses corresponding to some requests before logging in. For example, when you just opened the login page and sent a GET request, "csrftoken=..." appeared in the set-cookie of the response headers, and the value of csrftoken here is exactly the same as the value we need to submit in the login form. Therefore, we can get the value of csrfmiddlewaretoken by getting the Cookies in the previous response.

![8](https://raw.githubusercontent.com/chr1sc2y/warehouse-deprecated/refs/heads/main/resources/Use-Scrapy-to-Crawl-LeetCode/8.png)

First, we analyze the composition of Cookies by sending a GET request.
```
login_url = "https://leetcode.com/accounts/login/"
session = requests.session()
result = session.get(login_url)
print(result)
print(type(result.cookies))
for cookie in result.cookies:
    print(type(cookie))
    print(cookie)
```
The result is

- ```<Response [200]>```` status code 200, indicating that the request was successful
- ```<class 'requests.cookies.RequestsCookieJar'>``` The type of cookies is CookieJar
- ```<class 'http.cookiejar.Cookie'>``` The first cookie type is Cookie
- ```<Cookie__cfduid=d3e02d4309b848f9369e21671fabbce571548041181 for .leetcode.com/>``` The first cookie information
- ```<class 'http.cookiejar.Cookie'>``` The second cookie type is Cookie
- ```<Cookie csrftoken=13mQWE9tYN6g2IrlKY8oMLRc4VhVNoet4j328YdDapW2WC2nf93y5iCuzorovTDl for leetcode.com/>``` The second cookie information is the csrftoken we need

In this way, we obtain the csrfmiddlewaretoken needed when submitting form information, and then we can start writing the login-related code. By the way, the key of the csrf token automatically generated when using Django for back-end development is also called csrfmiddlewaretoken. I don’t know if LeetCode uses Django as the back-end development framework.

### Implementation

First, we need to obtain the login information before the crawler starts running, and save the Session as a member variable of the class, so that it can be used when obtaining submissions. At the same time, we need to create a new config.json in the same directory as the crawler file, and save our username and password in the json file, so that we can log in smoothly.
```
    def start_requests(self):
        self.Login()    # Log in
        questionset_url = "https://leetcode.com/api/problems/all/"
        yield scrapy.Request(url=questionset_url, callback=self.ParseQuestionSet)

    def Login(self):
    login_url = "https://leetcode.com/accounts/login/"
    login_headers = {
        "user_agent": "Mozilla/5.0 (Macintosh; Intel Mac OS X 10_14_2) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/71.0.3578.98 Safari/537.36'",
        "referer": "https://leetcode.com/accounts/login/",
        # "content-type": "multipart/form-data; boundary=----WebKitFormBoundary70YlQBtroATwu9Jx"
    }
    self.session = requests.session()
    result = self.session.get(login_url)
    file = open('./config.json', 'r')
    info = json.load(file)
    data = {"login": info["username"], "password": inf["password"],
            "csrfmiddlewaretoken": self.session.cookies['csrftoken']}
    self.session.post(login_url, data=data,headers=login_headers)
    print("login info: " + str(result))
```
Note that if you fill in the content-type value in the headers, some strange error messages may be generated, and your submissions cannot be obtained correctly later. Only the user_agent and refere information is needed.

If you see the output ```login info: <Response [200]>```, it means the login is successful!

## References

<a href="https://scrapy-chs.readthedocs.io/zh_CN/0.24/" target="_blank">Scrapy official documentation</a>
