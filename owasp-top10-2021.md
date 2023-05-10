# OWASP Top 10 - 2021
1. Broken Access Control
2. Cryptographic Failures
3. Injection
4. Insecure Design
5. Security Misconfiguration
6. Vulnerable and Outdated Components
7. Identification and Authentication Failures
8. Software and Data Integrity Failures
9. Security Logging & Monitoring Failures
10. Server-Side Request Forgery (SSRF)

# Broken Access Control
Websites have pages that are protected from regular visitors. For example, only the site's admin user should be able to access a page to manage other users. If a 
website visitor can access protected pages they are not meant to see, then the access controls are broken.

A regular visitor being able to access protected pages can lead to the following:
- Being able to view sensitive information from other users
- Accessing unauthorized functionality

Simply put, broken access control allows attackers to bypass authorisation, allowing them to view sensitive data or perform tasks they aren't supposed to.

For example, a vulnerability [was found in 2019](https://bugs.xdavidhu.me/google/2021/01/11/stealing-your-private-videos-one-frame-at-a-time/), where an attacker 
could get any single frame from a Youtube video marked as private. The researcher who found the vulnerability showed that he could ask for several frames and somewhat 
reconstruct the video. Since the expectation from a user when marking a video as private would be that nobody had access to it, this was indeed accepted as a broken 
access control vulnerability.

## Insecure Direct Object Reference
IDOR or Insecure Direct Object Reference refers to an access control vulnerability where you can access resources you wouldn't ordinarily be able to see. This occurs 
when the programmer exposes a Direct Object Reference, which is just an identifier that refers to specific objects within the server. By object, we could mean a 
file, a user, a bank account in a banking application, or anything really.

For example, let's say we're logging into our bank account, and after correctly authenticating ourselves, we get taken to a URL like 
this https://bank.thm/account?id=111111. On that page, we can see all our important bank details, and a user would do whatever they need to do and move along their 
way, thinking nothing is wrong.

![image](https://github.com/vivekprm/hackers-manual/assets/2403660/8f67170c-a0db-4ecb-94df-b62f7eb9788c)

There is, however, a potentially huge problem here, anyone may be able to change the id parameter to something else like 222222, and if the site is incorrectly 
configured, then he would have access to someone else's bank information.
![image](https://github.com/vivekprm/hackers-manual/assets/2403660/2256e0d1-4206-4ae0-a601-22b270b12d65)

The application exposes a direct object reference through the id parameter in the URL, which points to specific accounts. Since the application isn't checking if 
the logged-in user owns the referenced account, an attacker can get sensitive information from other users because of the IDOR vulnerability. Notice that direct 
object references aren't the problem, but rather that the application doesn't validate if the logged-in user should have access to the requested account.
