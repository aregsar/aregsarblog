# working with linux file permissions

[working with linux file permissions](https://aregsar.com/blog/2020/working-with-linux-file-permissions)

## Getting permissions info

list files and directories including their permissions

ls -l

include hidden files as well

ls -al

show permissions for a file or directory

ls -l filename_or_directory

## Changing the permissions of a file or directory

chmod permissions filename_or_directory

When we list the file or directory permissions with the ls command, they are divided into three ownership blocks in the following order:

[user-owner-permissions][group-owner-permissions][non-owner-permissions]

Each block has three bits associated with it. Each bit corresponds to a permission type in following order:

[rwx][rwx][rwx]

Where:

r=read permission bit
w=write permission bit
x=execute permission bit

When a the permission has a dash `-` instead of the corresponding permission letter, it means that it is not allowed.

So for example `rwxrw-r--` means the user that owns the file has all permissions, the group that owns the file only has read and write permissions and the non owner users only have read permissions.

To set the permissions to `rwxrw-r--` there are two general methods we can use.

The first is the using the octal number system:

```bash
chmod 764 somefile
```

This will set the file permissions to `rwxrw-r--` using the `111110100` which is the octal number system binary code for 764.

If we superimpose the permissions with the binary code we get:

```bash
rwxrw-r--
111110100
```

We can see that he zeros correspond to the dashes and the ones have the corresponding permission letter for the position.

The second way is to turn individual permissions on and off.

This turns on execute permission for somefile for user owner of file

chmod +x somefile

This does the opposite

chmod -x somefile

By changing the x to r or w above we can do the same for those permissions. or we use any combination of the three letters to turn on or off multiple permissions:

chmod -r somefile
chmod +rwx somefile
chmod +rx somefile

This turns on permission for somefile only for the group owner of file

chmod g+x somefile

This does the opposite

chmod g-x somefile

By changing the x to r or w above we can do the same for those permissions. or we use any combination of the three letters to turn on or off multiple permissions.

This turns on permission for somefile only for the non owner users

chmod o+x somefile

This does the opposite

chmod o-x somefile

By changing the x to r or w above we can do the same for those permissions. or we use any combination of the three letters to turn on or off multiple permissions.

## Changing the user and/or group ownership of a file or directory

change user and group owner of the file or directory

chown user:group filename_or_directory

change only the group of the file or directory

chown :group filename_or_directory

change only the user owner of the file or directory

chown user: filename_or_directory

the colon at the end is not required so we could do as well

chown user filename_or_directory

> Note since the chown command handles both users and groups I am not going to use the chgrp that only changes group ownership.

## Resources

https://www.guru99.com/file-permissions.html

https://www.pluralsight.com/blog/it-ops/linux-file-permissions
