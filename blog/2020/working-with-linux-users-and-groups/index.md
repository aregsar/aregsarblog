# working with linux users and groups

[working with linux users and groups](https://aregsar.com/blog/2020/working-with-linux-users-and-groups)

## Group Operations

add a new group

```bash
groupadd mynewgroup
```

example:

```bash
groupadd admin
```

## New User Operations using the useradd command

add a new user:

```bash
useradd -m newusername
```

the -m option creates a home directory of the same name

example:

```bash
useradd -m bob
```

When we create a new user a new group by the same name is created and the user is assigned to that group as its default group.

For the example above there will also be a new group named bob created and the user bob will be automatically assigned to this bob group as it default group.

Normally the new user id and the new group id will be the same but can get out of sync. One way to avoid this is to create a new group using `groupadd` and using the username as the name of the group then immediately follow that command by creating the user using `useradd` and assigning it to that group using the -g option:

```bash
groupadd bob && useradd -m -g bob bob
```

> Note: The -g option specified the default group to assign the user to. Using -G instead of -g will add the user to secondary group not the default group so we need to use lowercase character.

## Existing User Operations using the usermod command

Add existing user to existing group

```bash
usermod -aG examplegroup exampleusername
```

Example of adding user bob to the admin group

```bash
usermod -aG admin bob
```

Add existing user to multiple existing groups

```bash
usermod -aG group1,group2,group3 exampleusername
```

Example of adding user bob to both admin and sudo groups

```bash
usermod -aG admin,sudo bob
```

> Note sudo is a special group that allows the user to use the sudo command to elevate to root user privilages

Change the default group of a user to an existing group

```bash
usermod -g groupname username
```

example of changing the default group of user bob to the admin group

```bash
usermod -g admin bob
```

## Users Info

list all users on system

cat /etc/passwd

list all groups on system

cat /etc/group

show current user username

whoami

show group ids and default group for current user

id

show groups the current user belongs to

groups

show group ids and default group for a user

id username

show groups a user belongs to

groups exampleusername

on linux only `/etc/default/useradd`
