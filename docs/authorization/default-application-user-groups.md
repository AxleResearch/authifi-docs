## Automatically Assigning Users to Groups after Login

When a user logs in, they can be automatically assigned to one or more **default application user groups** configured for an application. This can be useful
for scenarios such as new user signups. Instead of having to manually assign new users to groups after they log in for the first time, this feature will automatically assign them to the designated default groups.

### Steps

- Create a new application or select an existing application.
- Create one or more User Groups.
- Assign the User Groups to the application default user groups. This action can be completed in the **Default User Groups** tab of the Auth UI or through the [Client REST APIs](https://a-ci.ncats.io/_api/auth/docs#tag/Clients).

When users login to the application, they will automatically become members of the configured groups. Assignment can be confirmed via the Auth Tenant REST APIs or by visiting the **Groups** dashboard on the Auth UI, selecting the group(s), and viewing users on the "Members" tab.
