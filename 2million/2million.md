# 2million

## Scanning
The following nmap scan applies default scripts and version detection to open ports, providing information on software versions and possible service vulnerabilities.

![image](https://github.com/KayEm06/HackTheBox/assets/62169414/aa4ee9b6-4b05-4241-b539-0572d530c3f1)

From exploring the web server hosted at port 80, we have identified two endpoints: `/login` and `/invite`. To expand our information on the web server, we can use `ffuf` to discover relevant endpoints by filtering out status code 301 (Moved Permanently) and 403 (Forbidden).

![image](https://github.com/KayEm06/HackTheBox/assets/62169414/177c9e80-30f8-49aa-91af-fdfcb9e3f5f6)

We began with the login page by entering possible default credentials, attempting SQL injection, and Cross-Site Scripting on the `error` query parameter, but with no success.

Looking at the source code of the invite page, one JavaScript program had an interesting filename (`/js/inviteapi.min/js`) and seemed to be obfuscated with regular expressions. We ran it through an online JavaScript beautifier to [deobfuscate](https://github.com/KayEm06/HackTheBox/blob/main/2million/inviteapi.js) and uncover two functions: `verifyInviteCode` and `makeInviteCode`. We attempted to generate an invite code by sending a POST request to `/api/v1/invite/how/to/generate`.

_(To send a POST request to the URL using curl you need to obtain your PHPSESSID found in the cookie storage of the web server. Alternatively, you can use burpsuite)_

![image](https://github.com/KayEm06/HackTheBox/assets/62169414/34720447-db8d-469d-ab3b-ee62eeaf0a77)

The `ffuf` scan had concluded, and no additional endpoints were discovered.

![image](https://github.com/KayEm06/HackTheBox/assets/62169414/ce5358ac-75ac-4e4a-bbc4-c78ab3bb3464)

Using an online ROT13 decrypter, the decrypted data read: "In order to generate the invite code, make a POST request to /api/v1/invite/generate" to which we sent a POST request.

![image](https://github.com/KayEm06/HackTheBox/assets/62169414/d1b76a84-2661-4b9f-baad-75e47dcb81f5)

The web server responded with a base64 encoded string that can be decoded by piping `base64 -d` to the string with echo on the command line or using an online base64 decoder. Invite Code: **JKIEK-XIHK7-6RG2Q-37HSH**

After backtracking to the invite page and entering the invite code, we were redirected to a registration page. Once registered, we were redirected to the home page made of mostly dummy endpoints. To get help with the challenge, I referred to a writeup created by @madfoxsec to find several endpoints at `/api/v1`. We decided to use burpsuite to speed up the enumeration of these endpoints.

![image](https://github.com/KayEm06/HackTheBox/assets/62169414/abc4d603-4466-439a-b429-d695dacd8117)

Of all endpoints, the subdirectories under `admin` sparked curiosity. We began with sending a GET to `/admin/auth` to verify if our user was admin. 

![image](https://github.com/KayEm06/HackTheBox/assets/62169414/5753085c-90f5-42c0-b5c2-c9be5fa67aa3)

The response message confirms our user is not an admin; however, it is possible to modify our privileges at `/admin/settings/update`. Sending a POST request to this URL requires your request to include the content type as json and an email parameter. `json` was selected as that was present in the response message from `/api/v1`.

![image](https://github.com/KayEm06/HackTheBox/assets/62169414/b5f369e8-f879-493d-b9cf-479d43fdedc1)

The response message shows an `is_admin` value of True for our username, informing us that our user is an admin. We confirmed this by sending another GET request to `/admin/auth`. 

![image](https://github.com/KayEm06/HackTheBox/assets/62169414/2050ae12-6512-4b8e-8977-7992bf666646)

Now that confirmed, we accessed the `/admin/vpn/generate` URL to test its functionality. We are asked by the web application to specify a username, after which an OpenVPN file is generated for the specified user. Since OpenVPN files are generated on the command line, we tested for interactivity on the web server's shell with a basic Proof of Concept payload such as `echo`.

![image](https://github.com/KayEm06/HackTheBox/assets/62169414/c5e2189a-7c47-4f7e-a6c7-ae483d1b7444)

The response message confirmed that we can. For greater control, we started a Netcat listener on our attacking machine to set up a reverse shell.

![image](https://github.com/KayEm06/HackTheBox/assets/62169414/9ab07226-bedb-44e7-a54b-c9dd0c761b7c)

On the web server's side, we connected to our attacking machine with a bash shell.

![image](https://github.com/KayEm06/HackTheBox/assets/62169414/6dd47a92-a3c9-4d03-a7d1-6772840d4e15)

We changed to our attacking machine's terminal to confirm we were connected.

![image](https://github.com/KayEm06/HackTheBox/assets/62169414/0ba0509a-41c2-4256-9ce4-806c50cea998)

After successfully setting up a reverse shell, we upgraded to a more stable shell.

![image](https://github.com/KayEm06/HackTheBox/assets/62169414/41acf416-0198-4e0a-8bbd-1ad8ef54e84f)

With the following combination of commands, we upgraded our shell to a fully interactive and stable one. We began enumerating the target machine with a simple `ls` command.

![image](https://github.com/KayEm06/HackTheBox/assets/62169414/09c3e2b5-41a9-4984-af20-e12df736a55a)

 We opened the `Database.php` file.

![image](https://github.com/KayEm06/HackTheBox/assets/62169414/c6f2f7c4-a35a-44bc-ad69-8466074d0904)

The file revealed four variables: `$host`, `$user`, `$pass`, and `$dbName`. Since most application variables are stored in the `.env` file, we determined if this file is present on the target machine with `ls -la`.

![image](https://github.com/KayEm06/HackTheBox/assets/62169414/bd2d7028-bc7c-45a6-af4c-b14092fb2846)

`.env` is indeed present, so we opened it.

![image](https://github.com/KayEm06/HackTheBox/assets/62169414/a9974643-1154-4140-b7b1-15a693b1492c)

Accessing the file gave us the following credentials: "admin": "SuperDuperPass123". We attempted to switch to the `admin` user. Switching to the admin user was successful, so we began enumerating the user by listing all the files they own.

![image](https://github.com/KayEm06/HackTheBox/assets/62169414/3c6450a1-3935-419f-b30f-cdc0cb0ca8ec)

We decided to start with the admin's home directory.

![image](https://github.com/KayEm06/HackTheBox/assets/62169414/e55cd549-f7df-4c69-b7c7-008dd0713d45)

From viewing the admin's home directory, we got the user flag. We can now view the `var/mail/admin` file.
**User flag: 280e5dc60091a860817b1a8b8f348445**

![image](https://github.com/KayEm06/HackTheBox/assets/62169414/f5908763-0f46-4eea-808a-450e218a22c1)

We learn from this email that the application is vulnerable to OverlayFS. We can search for any payloads around this vulnerability on GitHub: https://github.com/xkaneiki/CVE-2023-0386. We began by downloading the repository as a zip file and transferring it from our attacking machine to the target.

![image](https://github.com/KayEm06/HackTheBox/assets/62169414/07388b3d-7cf4-47fb-a304-ca7b95ee53b0)

Once transferred, we changed the directory to `/tmp` and unzipped the file.

![image](https://github.com/KayEm06/HackTheBox/assets/62169414/8c1ca90b-f2ad-4bd8-8bc0-78c275194454)

We followed the author's instructions with the command `make all`.

![image](https://github.com/KayEm06/HackTheBox/assets/62169414/91a2fdb8-bb3f-4f65-b44c-b6239726f55b)

Followed by `./fuse ./ovlcap/lower ./gc`

![image](https://github.com/KayEm06/HackTheBox/assets/62169414/966e1d28-f7c2-4c2f-891b-bcc10998711e)

Finally, `./exp`. We determined whether we have escalated our privileges with `whoami` and `id`.

![image](https://github.com/KayEm06/HackTheBox/assets/62169414/dba96fa8-0295-4f7c-bfc5-c7aa2ccf595e)

We successfully escalated our privileges to the root user and retrieved the root flag.

![image](https://github.com/KayEm06/HackTheBox/assets/62169414/a03c7307-cf72-4c65-9dbf-22f73b61cba5)

**Root flag: 5b288b17ffb742ecf0e2cb0ca38eb960**



