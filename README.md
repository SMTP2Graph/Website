# SMTP2Graph

> An open-source, robust, versatile and lightweight multiplatform SMTP server that relays messages over Microsoft 365/Exchange Online using the Microsoft Graph API.

| :warning: Under development                                                         |
|-----------------------------------------------------------------------------------------|
| SMTP2Graph is currently under development, but the first version is expected quite soon |

## What is it

SMTP2Graph is a SMTP server that will send messages over the Microsoft 365/Exchange Online platform. You don't need a userlicense for this, but you need to create an application registration in Entra ID (Azure AD) and assign it the desired permissions.

## Features

- SMTP AUTH support (PLAIN and LOGIN)
- TLS support
- IP whitelist
- FROM whitelist
- RCPT whitelist
- Rate limiter
- Brute force protection
