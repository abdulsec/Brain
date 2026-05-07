tags: #googlevrp #accepted #bounty #lithium

**Program:** Google VRP  
**URL:** [Google Cloud Community Profile](https://www.googlecloudcommunity.com/gc/user/myprofilepage/tab/personal-profile)  
**Vulnerability Type:** Privilege Escalation  

### Details

In the Google Cloud Community, users who register with an @google.com email address are automatically promoted to Google staff. Typically, users with non-Google emails do not receive such privileges. However, after registering with a non-Google email, users can change their email address to an @google.com domain in their profile settings without verification. This allows them to be automatically promoted to Google staff, bypassing the intended restrictions.

### Steps to Reproduce

1. Register a new account on Google Cloud Community using any non-@google.com email (e.g., example@gmail.com).
2. After registration, navigate to your profile settings under **My Profile**.
3. Change the registered email address to an @google.com email (e.g., abdulsec@google.com).
4. Upon saving the changes, observe that you are automatically promoted to a Google staff member without any verification process.

### Attack Scenario

This vulnerability allows unauthorized users to promote themselves to Google staff, which can lead to abuse of privileges and misrepresentation within the community. This exploit could be used to gain trust and access sensitive areas or mislead other community members.
