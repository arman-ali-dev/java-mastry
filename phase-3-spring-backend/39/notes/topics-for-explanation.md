1. Why We Can't Ask Users for Their Google Password
2. The Delegated Access Problem
3. What OAuth 2.0 Solves
4. Resource Owner
5. Client (Why It Doesn't Mean "Browser")
6. Authorization Server
7. Resource Server
8. Scope
9. Consent Screen
10. Access Token
11. Why OAuth Alone Isn't an Identity Protocol
12. OpenID Connect (OIDC)
13. ID Token
14. Important Claims (sub, iss, aud, exp)
15. Refresh Token
16. Access Token vs ID Token vs Refresh Token
17. Authorization Code Flow — Overview
18. client_id
19. redirect_uri
20. scope Parameter in the Request
21. response_type=code
22. state Parameter (CSRF Protection)
23. Why Google Credentials Never Reach Our Application
24. Authorization Code — What It Is and Isn't
25. Front Channel vs Back Channel
26. PKCE — What Problem It Solves
27. code_verifier and code_challenge
28. How the Authorization Server Verifies PKCE
29. Registering the Application with Google
30. Default Redirect URI Template
31. provider + providerSubject (Why Not Use Email)
32. Why AppUser Has No Password Field
33. CustomOidcUserService — Wrapping Built-in Processing
34. oauth2Login() Configuration
35. Why the Authorization Request Is Built by Backend but Sent by Browser
36. How the Authorization Code Reaches the Application
37. Session-Based Behavior After OAuth2 Login
38. oauth2Login() vs oauth2ResourceServer()
39. Why Authorization and Authentication Share the Same Flow
40. Complete End-to-End Flow Recap
