Deployment Scripts
==================

Assumptions about your codebase and server
==========================================
These scripts assume the following facts about your project and deployment:

* You are hosting the code you wish to use in the deployment on a git repo accessible from the server
* The server is running Ubuntu (preferably 10.10 but should work with most of the newer flavours)
* You have an account that can ssh into the server with sudo powers
* This is a lone deployment. Restarts to Apache/other procs can happen and shouldn't affect other unrelated deployments (i.e. the fabric script will restart apache server/supervisor procs at will)

Requirements
=============
The files in this repo.
Supervisor
Apache2
(the above two packages will be attempted to be installed automatically by the fab setup_server command, but may require additional manual setup)

Installation
============
These scripts should be checked out to a seperate (from your development repo) folder and copied into your development library.  Unfortunately this results in you losing the ability to automatically get the latest updates to this repo in your deployment.  However, you will need to make some project-specific edits to these files plus likely add some custom code to the fabfile (for specific processes that need to be launched, etc).  Consider these files a good starting framework for a full deployment harness.

That said, you should be able to do your first deployment with a minimal amount of editing to the below files.

List of files

* **fabfile.py** - The fabric file responsible for initial setup and deployment activities.  Should be configured to hold the relevant IP, hostname, usernames and branches necessary for deployment.
* **services/templates/apache.conf** - The apache template used for hosting staticfiles and proxying traffic from port 80 to/from the gunicorn/cpserver process
* **services/templates/supervisord.conf** - The supervisor template that monitors the updtime/status of the underlying service processes (gunicorn, celery, router, etc)

Both of the templates listed above are populated by the fabric script at deploy time.  See either file for example usage (makes use of python string replacement).   The full configuration file is generated locally and then scp'd over to the live machine (NOT copied from the remotely checked out repo).  Once the files are in your own repo, feel free to edit them to include custom services/apache configurations that you would like to see.
**WARNING:** do not make changes to apache/supervisor confs on the remote machine.  They will be overriden by a freshly template generated config on each deploy command.

* **requirements/** - Text files listing the apt-get packages and pip require packages needed for the server.   Edit as needed.

Usage
=====
(For a full list of commands that can be performed by the fabric script, run fab -l from the same location as the fabfile.py)

Initial deployment on a clean (untouched) machine:
    fab production setup_server bootstrap deploy

Standard deployment:
    fab production deploy

(above commands assume production environment.  For staging, use "staging" instead)

The script assumes an account with the same username as local exists on the server.  For example if I run the script under adewinter, the fab script will attempt to log in to the remote machine with the username: adewinter.  
If you would like to use a different username (for example a shared account on the staging/production machine), use
    fab --user=foo production ...
where "foo" is the username you would like to log in as.


Server Deployment Structure
===========================
There are some key points to take into consideration when using this deployment bundle.  The fabric file aims to deploy the Django project in a standardized way that is the same across machines/projects in order to allow anyone from Dimagi familiar with the process to dive in immediately and diagnose a problem should an issue arise.

Folder Structure:

`
	.                   // . is /home/<PROJECT_NAME>/
	|-- services
	|   |-- apache      //home for apache conf
	|   `-- supervisor  //home for supervisor conf
	`-- www
		|-- staging
			|-- code_root  //actual project code lives here
			|-- log        //*all* project related logs go here
			`-- python_env  //home for the virtualenv for this project
`

The services confs are usually symlinked to the correct place in-system.  For example, the apache conf located in /home/myproject/services/apache/myproject_apache.conf will be symlinked to the /etc/apache2/sites-enabled/ directory.

Every time a deploy occurs, the apache config file will be touched (and/or an apache reload/restart will be triggered), to ensure the latest settings are actually being used.  Similarly a 'sudo supervisorctl update' will be issued for any supervisor conf updates.


Aux Info
========
These production deployment scripts assume a Linux-like environment (as in Ubuntu Linux, Cygwin/Mingw32 on Windows, etc) primarily due to an issue with path seperators (we use posixpath to get around this, see bug-ticket: http://code.fabfile.org/attachments/61/posixpath.patch)

