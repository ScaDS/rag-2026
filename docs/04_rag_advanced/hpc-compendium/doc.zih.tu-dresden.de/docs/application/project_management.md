# User Management for Project Leaders

The HPC project leader (PI/PC) has overall responsibility for the project and for all activities
within the corresponding project on ZIH systems. In particular the project leader shall:

* add and remove users from the project,
* update contact details of the project members,
* monitor the resources of the project,
* inspect and store data of retiring users.

The project leader can appoint a *project administrator* with an HPC account to manage these
technical details.

The [project management site](#access) enables the project leader and the project administrator
to

* get a [project overview](#projects)
* [add and remove users from the project](#manage-project-members-dis-enable)
* [define a technical administrator](#manage-project-members-dis-enable)
* [view statistics (resource consumption)](#statistic)
* file a new HPC proposal,
* file results of the HPC project.

## Access

[Entry point to the project management system](https://hpcprojekte.zih.tu-dresden.de/managers)
The project leaders of an ongoing project and their accredited admins
are allowed to login to the system. In general each of these persons
should possess a ZIH login at the Technical University of Dresden, with
which it is possible to log in to the website. In some cases, it may
happen that a project leader of a foreign organization does not have a ZIH
login. For this purpose, it is possible to set a local password:
"[Missing Password](https://hpcprojekte.zih.tu-dresden.de/managers/members/missingPassword)".

![Login Screen](misc/external_login.png "Login Screen")
{: align="center"}

On the 'Missing Password' page, it is possible to reset the passwords of a 'non-ZIH-login'. For this
you write your login, which usually corresponds to your e-mail address, in the 'Login' field and
click on 'reset'. Within 10 minutes the system sends a signed e-mail from
<hpcprojekte@zih.tu-dresden.de> to the registered e-mail address. this e-mail contains a link to
reset the password.

![Password Reset](misc/password.png "Password Reset")
{: align="center"}

## Projects

After login you reach an overview that displays all available projects. In each of these listed
projects, you are either project leader or an assigned project administrator. From this list, you
have the option to view the details of a project or make a follow-up application (extension). The
latter is only possible if a project has been approved and is active or was. In the upper right
area you will find a red button to log out from the system.

![Project Overview](misc/overview.png "Project Overview")
{: align="center"}

The project details provide information about the requested and allocated resources. The other tabs
show the employee and the statistics about the project.

![Project Details](misc/project_details.png "Project Details")
{: align="center"}

### Manage Project Members (dis-/enable)

The project members can be managed under the tab 'employee' in the project details. This page gives
an overview of all ZIH logins that are a member of a project and its status. If a project member
marked in green, it can work on all authorized HPC machines when the project has been approved. If
an employee is marked in red, this can have several causes:

* the employee was manually disabled by project managers, project administrator
  or ZIH staff
* the employee was disabled by the system because its ZIH login expired
* confirmation of the current HPC-terms is missing

You can specify a user as an administrator. This user can then access the project management system.
Next, you can disable individual project members. This disabling is only a "request of disabling"
and has a time delay of 5 minutes. A user can add or reactivate itself within a specific project by
clicking the link on the end of the page. To prevent misuse this link is valid for 2 weeks and
will then be renewed automatically.

![Project Members](misc/members.png "Project Members")
{: align="center"}

The link leads to a page where you can sign in to a project by accepting the terms of use. You also
need a valid ZIH-Login. After this step it can take 1-1.5 h to transfer the login to all cluster
nodes.

![Add Member](misc/add_member.png "Add Member")
{: align="center"}

### Statistic

The statistic is located under the tab 'Statistic' in the project details. The data will be updated
once a day and shows used CPU-time and used disk space of a project. Follow-up projects also show
the data of their predecessor(s).

![Project Statistic](misc/stats.png "Project Statistic")
{: align="center"}
