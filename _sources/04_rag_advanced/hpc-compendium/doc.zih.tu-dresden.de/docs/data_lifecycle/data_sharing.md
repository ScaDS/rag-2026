# Sharing Data

This page should provide you some commands to share your data with other users or projects.

## Grant access on some file or directory to persons in your project

If all persons that should be able to access your data are in the same project, you can give them
access to your workspace, e. g. `input-data` via the following commands:

```console
marie@login$ id --group --name
p_number_crunch
marie@login$ chown -R marie:p_number_crunch /scratch/ws/1/marie-input-data
```

Now, everyone who is in project `p_number_crunch` should be able to access your data. If this is not
the case, you should check whether the file that your colleague wants to access is readable for the
group (`r` permission is set for the group) and every parent directory of that file is searchable
for the group (`x` permission is set for the group). For example, in the following case, a colleague
of `marie` cannot access `data-file` because the base directory `.` is not searchable for the group
as it does not have the `x` permission, even though the file has the permission `r` set for the
group. Thus, `marie` has to make the directory searchable by using `chmod`:

```console
marie@login$ ls -la /scratch/ws/1/marie-input-data
drwxr-----   4 marie    p_number_crunch   4096 27. Jun 17:13 .
drwxr-xr-x 444 operator adm             151552 14. Jul 09:41 ..
-rw-r-----   2 marie    p_number_crunch   4096 27. Jun 17:13 data-file
marie@login$ chmod g+x /scratch/ws/1/marie-input-data
marie@login$ ls -la /scratch/ws/1/marie-input-data
drwxr-x---   4 marie    p_number_crunch   4096 27. Jun 17:13 .
drwxr-xr-x 444 operator adm             151552 14. Jul 09:41 ..
-rw-r-----   2 marie    p_number_crunch   4096 27. Jun 17:13 data-file
```

!!! danger "New file inherits group and permissions of the creator"

    When a user creates a file, the created file is associated to that user and inherits the user's
    default group. If the user is in multiple groups/projects, he/she has to ensure, that the new
    file is associated with the project's group. This can be done using `chown` and `chmod` as shown
    above. Another possibility is to use an environment file `env.sh` with the following content:

    ```bash
    newgrp p_number_crunch  # files should have this group by default
    umask o-rwx             # prevent creating files that allow persons not in this group (a.k.a. others) to read, write or execute something
    ```

    Before creating new files, users can now load this file using `source` in order to ensure that
    new files automatically get the right group:

    ```console
    marie@login$ cd /scratch/ws/1/marie-input-data
    marie@login$ source /projects/p_number_crunch/env.sh
    bash-4.2$ touch new-file    #create a new file
    ```

Read on, if you want to restrict access to specific persons outside of your group, but don't want to
permit everyone to access your data.

## Grant access on some file or directory to persons from various projects

[Access Control Lists](https://en.wikipedia.org/wiki/Access-control_list) (ACLs) can be used, when
`chmod` is not sufficient anymore, e. g. because you want to permit accessing a particular file for
persons from your project and also some persons outside of your project, but not everyone.

!!! note

    At the moment `setfacl` is only working on our Lustre filesystems (`/data/horse` and
    `/data/walrus`).

The command `setfacl` is used to manage access rights for workspaces. To view the current access
rights, use the command `getfacl`. The following commands are used to grant a user access to the
workspace.

If you are unsure what your group/project is, you can use the following command to find out:

```console
marie@login$ id --group --name
p_number_crunch
```

If you are in multiple projects, you could see all of them using `--groups` instead of `--group`:

```console
marie@login$ id --groups --name
p_number_crunch
```

!!! example "Grant a user full access to the workspace folder"

    ```console
    marie@login$ setfacl --modify=u:<username>:rwx <path_to_workspace>
    ```

    For example, if `marie` wants to give her colleague `martin` access to the workspace
    `input-data` she has created, she would use the following command:

    ```console
    marie@login$ setfacl --modify=u:martin:rwx /scratch/ws/1/marie-input-data
    ```

!!! example "Inherit these same rights to all newly created files and folders"

    ```console
    marie@login$ setfacl --modify=d:u:<username>:rwx <path_to_workspace>
    ```

!!! example "Grant a project full access to the workspace folder"

    ```console
    marie@login$ setfacl --modify=g:<projectname>:rwx <path_to_workspace>
    ```

    For example, if `marie` wants to give all colleagues in `martin`'s project `p_long_computations`
    access to the workspace `input-data` she has allocated, she would use the following command:

    ```console
    marie@login$ setfacl --modify=g:p_long_computations:rwx /scratch/ws/1/marie-input-data
    ```

!!! example "Inherit these same rights to all newly created files and folders"

    ```console
    marie@login$ setfacl --modify=d:g:<projectname>:rwx <path_to_workspace>
    ```

If you already have files inside your workspace, remember to use the `-R` or `--recursive` options
to apply these ACL changes to all files.

!!! example "Remove access rights for a particular user"

    If you want to remove a user's access then use:

    ```console
    marie@login$ setfacl --remove=u:<username> <path_to_workspace>
    ```

    Remember to also remove the default access rights, if you added them previously:

    ```console
    marie@login$ setfacl --remove=d:u:<username> <path_to_workspace>
    ```

    For example, if `marie` wants to remove access to the workspace `input-data` she has given
    `martin` earlier:

    ```console
    marie@login$ setfacl --remove=u:martin /scratch/ws/1/marie-input-data
    ```

    And just to be sure, she would also remove default access rights for him:

    ```console
    marie@login$ setfacl --remove=d:u:martin /scratch/ws/1/marie-input-data
    ```

More details on ACLs can be found on the [setfacl man page](https://man.archlinux.org/man/setfacl.1).
