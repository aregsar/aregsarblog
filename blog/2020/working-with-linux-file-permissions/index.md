# working with linux file permissions

[working with linux file permissions](https://aregsar.com/blog/2020/working-with-linux-file-permissions)

## Getting permissions info

list files and directories including their permissions

ls -l

include hidden files as well

ls -al

show permissions for a file or directory

ls -l filename_or_directory

## Changing the permissions of a file or directory using chmod

We can change the permissions for a files and directories using he chmod command which stands for change mode.

chmod permissions filename_or_directory

If we are changing the permissions for a directory and we want the changes to be applied recursively for all subfiles and subfolder we can add the -R option

chmod -R permissions directory

### Permissions classes and types

When we list the file or directory permissions with the ls command, they are divided into three ownership classes in the following order:

[user-owner-permissions][group-owner-permissions][non-owner-permissions]

Each class has three bits associated with it, where each bit corresponds to a permission type in following order:

[rwx][rwx][rwx]

The rwx permission types are:

r=read permission
w=write permission
x=execute permission

For each position, when a permission bit has a dash `-` instead of the permission type, it means that the corresponding permission type is turned off. Otherwise the permission type in that position is on.

So for example `rwxrw-r--` means the user that owns the file has all permissions, the group that owns the file only has read and write permissions and the non owner users class only have read permissions.

### Setting the permissions

There are two ways to set the permission types.

One way uses the octal number system to specify which permission bits are turned on or off.

Below is an example the octal number system is used to set the permissions:

```bash
chmod 764 somefile
```

This will set the file permissions to `rwxrw-r--` using the binary number `111110100` which is the octal number system binary code for 764.

If we superimpose the permissions with the binary code we get:

```bash
rwxrw-r--
111110100
```

We can see that the zeros correspond to the dashes and the ones correspond to the permission type for the position.

The other way to set permissions, uses permission class letters and permission type letters and symbols to turn the the permission bits on or off. It is described in the following section.

### Setting permissions using permission class, permission type and symbols

The general form of the other way to set the permissions is:

chmod <permission class(s) to be changed><symbol><permission type(s) to be changed>

Or more specifically:

chmod <ugo|a|none><symbol><rwx|none>

The symbol can be one of `+`, `-` or `=`.

The permission class to be changed can be any combination of `ugo` with one, two, all or none being present. If they are all present, then an `a` can be used instead, signifying all permission classes will be changed. When none are present, it also signifies that all permission classes will be changed.

The permission type to be changed can be any combination of `rwx` with at least one, two or all being present when using the `+` or `-` symbol. When using he `=` symbol, one, two, all or none can be present. When the `-` or `=` symbol is used, the permission types that are not specified will set that permission type to a dash `-` for the corresponding permission type of the classes that are having their permission changed.

See the links in the resources for examples of changing user permissions.

## Changing the user and/or group ownership of a file or directory using chown

We can change the ownership of a file or directory using the `chown` command which stands for change owner.

change only the user owner of the file or directory

chown user filename_or_directory

change user and group owner of the file or directory

chown user:group filename_or_directory

change only the group of the file or directory

chown :group filename_or_directory

This final option changes the user and changes the group to the user's default group:

chown user: filename_or_directory

With this option there is a colon appended to the user name and no group name is specified.

> If you only want to change a users group, there is also a chgrp command that can be used instead of chown

## Resources

https://en.wikipedia.org/wiki/Chmod

https://www.guru99.com/file-permissions.html

https://www.pluralsight.com/blog/it-ops/linux-file-permissions
