# ✅ **Privilege Escalation & Business Logic Testing Checklist for Zoom Role Management**

### **1️⃣ General Privilege Escalation Testing**

1. Attempt to escalate privileges by modifying role assignments.
2. Try to add a higher-privilege role to yourself.
3. Remove restrictions from an existing role.
4. Check if lower-privileged users can edit higher-privileged roles.
5. Attempt to delete an admin role.
6. Assign an admin role to a user without permission.
7. Create a new role with admin-like permissions without being an admin.
8. Try to grant yourself **Billing, Security, or SSO settings** access.
9. Check if **role changes are logged properly** (audit logs).
10. Look for **role duplication issues** (cloning a high-privileged role).

### **2️⃣ Role Creation & Management**

1. Attempt to create a role with unexpected permissions.
2. Try to bypass role restrictions during creation.
3. Assign a role to a non-existent user.
4. Assign multiple conflicting roles to a user.
5. Try to modify a system-created role.
6. Delete a critical role and check its effect.
7. Look for **role caching issues** (stale roles being used).
8. Try to **import/export role configurations** for security flaws.
9. Look for **cross-tenant role assignment issues**.
10. Check if a role deletion affects **active sessions**.

### **3️⃣ API-Based Privilege Escalation**

11. Modify role assignments via API.
12. Bypass permission checks in API requests.
13. Look for **Rate Limiting issues** in role APIs.
14. Modify a role ID to escalate privileges.
15. Try to modify a user's role without authentication.
16. Look for **insecure direct object references (IDOR)** in role management API.
17. Check if API endpoints allow **modifying admin privileges**.
18. See if API requests return **excessive data** about roles.
19. Check for **Race Conditions** when updating roles.
20. Use **burp intruder** to brute-force role parameters.

### **4️⃣ Business Logic Flaws in User & Permission Management**

21. Try to **link an external user** as an admin.
22. Assign an inactive or deleted user a new role.
23. Reassign a banned user a new role.
24. Check if users can edit **their own permissions**.
25. Assign a free user a **paid plan role**.
26. Reuse old authentication tokens to modify roles.
27. Try to approve role changes without permissions.
28. Change a user’s permissions **through an indirect API call**.
29. Try to **reassign an account owner** through role changes.
30. Test what happens if a user **has no assigned role**.

### **5️⃣ Role-Based UI Manipulation**

31. Hide UI elements but still perform actions via API.
32. Look for **misaligned UI elements** for role-based access.
33. Check if restricted buttons **become clickable via dev tools**.
34. Inspect **hidden fields in HTML** for role information.
35. Try role changes via **JavaScript Console Injection**.
36. Modify role values in **local storage or cookies**.
37. Look for **client-side validation bypasses** in role assignments.
38. Use **Tamper Data/Fiddler** to modify role-related requests.
39. Check for missing **role descriptions and warnings**.
40. Inspect **misconfigured session handling** for roles.

### **6️⃣ Business Logic Attacks in Account Management**

41. Modify account settings as a non-admin.
42. Remove a required field in role changes.
43. Delete a high-privilege role and see if it gets reassigned.
44. Test session hijacking after role changes.
45. Create a duplicate admin user via role changes.
46. Assign a suspended user a valid role.
47. Test **long session expiration times** for admins.
48. Check if bulk role changes **lock out all admins**.
49. Try **cross-account role manipulations**.
50. Test if **OAuth tokens respect role changes**.

### **7️⃣ Device Management & Role Escalation**

51. Assign a role that allows managing encrypted devices.
52. Change a user’s device settings via a lower privilege role.
53. Modify device encryption settings without permission.
54. Bypass **Device Management UI restrictions**.
55. Look for missing **role-specific logging** in Device Management.
56. Try modifying **another user’s device settings**.
57. See if **role-based encryption keys** can be accessed.
58. Test **session persistence** when removing device access.
59. Disable security policies using an unintended role.
60. Modify **MFA settings for another user’s devices**.

### **8️⃣ Billing & Financial Abuse**

61. Try adding paid features via a low-privilege role.
62. Assign yourself **billing access** via role escalation.
63. Modify **subscription details** using a normal user account.
64. Delete **payment history** via misconfigured roles.
65. Bypass **billing notifications** via permission abuse.
66. Change **invoicing email recipients** via role manipulation.
67. Upgrade an account without valid credentials.
68. Try issuing refunds without permission.
69. Test what happens when an admin **removes all billing users**.
70. Modify **trial account settings** via privilege escalation.