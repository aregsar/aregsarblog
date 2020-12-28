# working with linux users and groups

December 19, 2020 by [Areg Sarkissian](https://aregsar.com/about)

[working with linux users and groups](https://aregsar.com/blog/2020/working-with-linux-users-and-groups)

In this post I show how the `groupadd`, `useradd` and `usermod` commands can be used to create and manage users and groups.

In addition will mention the `adduser` command that is an alternative to using the `useradd` command.

I will also describe the `su` command used to switch users.

Finally I will also show commands to show information about users and groups in the system.

## Group Operations

add a new group

```bash
groupadd mynewgroup
```

example:

```bash
groupadd admin
```

> Note there is also a higher level addgroup command on some systems that performs the same function as groupadd but in an interactive manner. So it is not well suited for scripting.

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

> Note: When using the useradd command we can not set the password using the -p option of the command. This is because that option expects us to provide the encrypted password. Therefor we must set the password explicitly after we create the user.

When we create a new user a new group by the same name is created and the user is assigned to that group as its default group.

For the example above there will also be a new group named bob created and the user bob will be automatically assigned to this bob group as it default group.

Normally the new user id and the new group id will be the same but can get out of sync. One way to avoid this is to create a new group using `groupadd` and using the username as the name of the group then immediately follow that command by creating the user using `useradd` and assigning it to that group using the -g option:

```bash
groupadd bob && useradd -m -g bob bob
```

> Note: The -g option specified the default group to assign the user to. Using -G instead of -g will add the user to secondary group not the default group so we need to use lowercase character. One the other hand if we did want to create a user with an automatic default group and assign it to a secondary group an example would be `useradd -m -G admin bob` where we create the user bob that gets a default group of bob and is also added to the existing admin group.

## New User Operations using the adduser command

The adduser command is a higher level command that uses the lower level useradd command in combination of other commands to add a new user with a home directory and password using a simpler interface. Unlike the useradd command, it requires user interaction to set a password for the added user.

example:

```bash
adduser bob
```

This command will add the user bob along with a default group named bob and will add a home directory for the user bob copying a set of default home directory files. It will also set a password for the user by asking you to type in a password.

The interactivity of adduser makes it un-suitable for scripting but we can make it non interactive with the following command:

```bash
adduser --disabled-password --gecos "" bob
```

But now we will need to set the password using the passwd command that is described in the next section.

Note: --disabled-password will create a user with no password, not a user with an empty password. The distinction is that the user with --disabled-password can not login to the system with any password whereas the user with an empty password can log in without a password.

## Setting the password for user

After we create a user without a password we can create a password for it using the passwd or chpasswd commands:

Since Both commands ask for interactive input for password and password confirmation, we can use the following commands scripts to make them non interactive:

```bash
echo -e "bobspassword\nbobspassword" | passwd bob
```

Alternate method:

```bash
echo "bobspassword" | passwd bob --stdin
```

Or using chpasswd

```bash
echo 'bob:bobspassword' | chpasswd
```

We can also make the user have an empty password by deleting the password:

```bash
passwd -d bob
```

> Note The -d option is not the same as `passswd -l bob` which locks the password for bob and is the same as the specifying the `--disabled-password` option for `adduser` command when creating user bob.

## Existing User Operations using the usermod command

Add existing user to existing group

```bash
usermod -aG examplegroup exampleusername
```

> Note if the `a` is removed then it will replace the current secondary groups the user belongs to with the given group instead of appending the group.

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

> Note sudo is a special group that allows the user to use the sudo command to elevate to root user privileges.

Change the default group of a user to another existing group

```bash
usermod -g groupname username
```

example of changing the default group of user bob to the admin group

```bash
usermod -g admin bob
```

## Switching users

WE can switch from the current user logged into the system to a different existing user by using the su (switch user) command:

```bash
su otheruser
```

We can go back to the user we switched from by typing `exit`. Note this can be done through multiple levels of `su` to get back to the original logged in user.
After that if we type `exit` once more we will get logged out.

example we are logged in as bob:

```bash
#switch from bob to bill
su bill
#switch from bill to jill
su jill
#go back to bill
exit
#go back to bob
exit
#logout bob
exit
```

## Users Info

show current user username

```bash
whoami
```

show current users username and id, default group name and id, and all groups for current user

```bash
id
```

show current users id

```bash
id -u
```

show current users default group id

```bash
id -g
```

show groups names of the groups the current user belongs to

```bash
groups
```

show username and id, default group name and id, and all groups for given user

```bash
id exampleusername
```

show groups given user belongs to

```bash
groups exampleusername
```

list all users on system

```bash
cat /etc/passwd
```

list all groups on system

```bash
cat /etc/group
```

## Resources

[https://linuxize.com/post/how-to-create-users-in-linux-using-the-useradd-command](https://linuxize.com/post/how-to-create-users-in-linux-using-the-useradd-command)
