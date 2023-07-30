# 2million

The following nmap scan applies default scripts and version detection to open ports, providing information on software versions and possible service vulnerabilities.

![image](https://github.com/KayEm06/HackTheBox/assets/62169414/aa4ee9b6-4b05-4241-b539-0572d530c3f1)

We can access the website hosted on port 80 and explore the website. Whilst exploring, we can run ffuf to fuzz any directories.

![image](https://github.com/KayEm06/HackTheBox/assets/62169414/177c9e80-30f8-49aa-91af-fdfcb9e3f5f6)

Of all the endpoints, two stand out: `/login` and `/invite`. I begin with the login page by entering possible default credentials, attempting SQL injection, and Cross-Site Scripting on the `error` query parameter, but with no success.

Looking at the source code of the invite page, one JavaScript program has an interesting filename (`/js/inviteapi.min/js'). The program seems to be obfuscated with regular expression so we can run it through an online JavaScript beautifier. The [deobfuscated program](https://github.com/KayEm06/HackTheBox/blob/main/2million/inviteapi.js) shows two used to verify and generate invite codes. We can attempt to generate an invite code by sending a POST request to `/api/v1/invite/how/to/generate`

_(To send a POST request to the URL using curl you will need to obtain your PHPSESSID which can be found in the cookie storage of the website. Alternatively, you can use burpsuite)_
![image](https://github.com/KayEm06/HackTheBox/assets/62169414/34720447-db8d-469d-ab3b-ee62eeaf0a77)

We can use an online ROT13 decrypter to get: "In order to generate the invite code, make a POST request to /api/v1/invite/generate"

_(At this point the ffuf scan has completed and found zero directories.)_
![image](https://github.com/KayEm06/HackTheBox/assets/62169414/ce5358ac-75ac-4e4a-bbc4-c78ab3bb3464)

![image](https://github.com/KayEm06/HackTheBox/assets/62169414/d1b76a84-2661-4b9f-baad-75e47dcb81f5)

Invite code: JKIEK-XIHK7-6RG2Q-37HSH

We can backtrack to the invite page and enter our invite code, which redirects us to a register page. Once registered, we are redirected to the home page and can begin exploring. Most of the endpoints are dummy endpoints so I refer to a writeup made by @madfoxsec to find several endpoints at `/api/v1`. I load up burpsuite to speed up the process of accessing all endpoints.

![image](https://github.com/KayEm06/HackTheBox/assets/62169414/abc4d603-4466-439a-b429-d695dacd8117)

![image](https://github.com/KayEm06/HackTheBox/assets/62169414/da7aec5d-a4f2-4326-8f06-519955cd4cbc)

![image](https://github.com/KayEm06/HackTheBox/assets/62169414/d392fe6a-a566-4ebe-b18f-c3a06721b1ca)

![image](https://github.com/KayEm06/HackTheBox/assets/62169414/a9690fb1-cda5-45ef-8d71-21bcbead538d)

![image](https://github.com/KayEm06/HackTheBox/assets/62169414/c5e2189a-7c47-4f7e-a6c7-ae483d1b7444)

![image](https://github.com/KayEm06/HackTheBox/assets/62169414/9ab07226-bedb-44e7-a54b-c9dd0c761b7c)

![image](https://github.com/KayEm06/HackTheBox/assets/62169414/6dd47a92-a3c9-4d03-a7d1-6772840d4e15)

![image](https://github.com/KayEm06/HackTheBox/assets/62169414/0ba0509a-41c2-4256-9ce4-806c50cea998)

![image](https://github.com/KayEm06/HackTheBox/assets/62169414/41acf416-0198-4e0a-8bbd-1ad8ef54e84f)

![image](https://github.com/KayEm06/HackTheBox/assets/62169414/09c3e2b5-41a9-4984-af20-e12df736a55a)

![image](https://github.com/KayEm06/HackTheBox/assets/62169414/c6f2f7c4-a35a-44bc-ad69-8466074d0904)

![image](https://github.com/KayEm06/HackTheBox/assets/62169414/bd2d7028-bc7c-45a6-af4c-b14092fb2846)

![image](https://github.com/KayEm06/HackTheBox/assets/62169414/a9974643-1154-4140-b7b1-15a693b1492c)

![image](https://github.com/KayEm06/HackTheBox/assets/62169414/3c6450a1-3935-419f-b30f-cdc0cb0ca8ec)

![image](https://github.com/KayEm06/HackTheBox/assets/62169414/e55cd549-f7df-4c69-b7c7-008dd0713d45)

![image](https://github.com/KayEm06/HackTheBox/assets/62169414/f5908763-0f46-4eea-808a-450e218a22c1)

![image](https://github.com/KayEm06/HackTheBox/assets/62169414/07388b3d-7cf4-47fb-a304-ca7b95ee53b0)

![image](https://github.com/KayEm06/HackTheBox/assets/62169414/8c1ca90b-f2ad-4bd8-8bc0-78c275194454)

![image](https://github.com/KayEm06/HackTheBox/assets/62169414/91a2fdb8-bb3f-4f65-b44c-b6239726f55b)

![image](https://github.com/KayEm06/HackTheBox/assets/62169414/966e1d28-f7c2-4c2f-891b-bcc10998711e)

![image](https://github.com/KayEm06/HackTheBox/assets/62169414/77126bed-2638-4b93-8d7f-ade18ff9c1a0)



