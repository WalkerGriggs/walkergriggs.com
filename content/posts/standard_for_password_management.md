+++
title = "A Standard for Password Management"
author = ["Walker Griggs"]
date = 2021-12-06
categories = ["devlog"]
draft = true
creator = "Emacs 27.2 (Org mode 9.4.4 + ox-hugo)"
weight = 2005
+++

## tldr; {#tldr}

Passwords are inherently insecure. We've layered a number of secure practices (some consumer facing, others system facnig) like MFA, security questions, oauth, and OIDC to complimen passwords and have built supporting systems like password managers to enable users to reliably and safely use sufficiently secure passwords, but we haven't written a standard for password management.

I propose a standard set of endpoints which let users, or password managers by proxy, programatically manage their passwords.

Use case: Say a user has 100 accounts at 100 different websites. Some, but not all support MFA. The user wants to rotate their passwords semi-regularly. Currently, they have to visit each of the 100 websites, login, navigate through unique account settings,  manually update their password, and update their password manager.

Instead, a user should be able to press one button in their password manager which will programatically generate a new password and update the account settings through the proposed endpoints. Better yet, the password manager should do this automatically every N days without the user needing to trigger the process.
