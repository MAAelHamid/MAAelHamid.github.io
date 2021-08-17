---

title: Bypass 30X redirect with BurpSuite
date: 2021-08-17 05:00:00 +0200
categories: [Tips and Tricks]
tags: [30x, burpsuite, bypass_redirection] 

---

HTTP response code **302 Found** is a common way to preform redirection 

Request

```http
GET /dashboard.php	HTTP/1.1
Host: corp.com
```

Response


```http
HTTP/1.1 302 Found
location: http://corp.com/loging.php
```

if we change `302 Found` to`200 OK` may be able to access dashboard without login

---

# Automate this process with BurpSuite

- Proxy -> Options -> Intercept Server Responses -> Check box (Intercept responses…)

![img](/assets/2021-08-17-Bypass-30X-redirect-with-BurpSuite.assets/word-image-266.png)

- edit “Match and Replace” section

  ![img](/assets/2021-08-17-Bypass-30X-redirect-with-BurpSuite.assets/word-image-267.png)

- Add

  ![img](/assets/2021-08-17-Bypass-30X-redirect-with-BurpSuite.assets/word-image-268.png)

- Fill in the blanks
  - Type: Response header

  - Match: 30[12] Found #match either 301 or 302

  - Replace: 200 OK

  - Comment: 30[12] Found bypass

  - Check “Regex match”

    ![image-20210817143003254](/assets/2021-08-17-Bypass-30X-redirect-with-BurpSuite.assets/image-20210817143003254.png)

- Click OK, enable the setting by activating the checkbox

  ![img](/assets/2021-08-17-Bypass-30X-redirect-with-BurpSuite.assets/word-image-270.png)

