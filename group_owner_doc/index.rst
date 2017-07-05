.. pebbles group admin documentation master file, created by
   suvileht 2017-07-04

=================================
Pebbles Group Owner Documentation
=================================

This document is intended to be for the group owners in Pebbles.

This document aims to be as brief as possible and link to relevant parts of
the more comprehensive documentation and relevant artefacts.

Terminology
===========

**group**
    is a rights management unit: the regular members of a group have
    access to the same set of **blueprints**

**blueprints**
    are a set of instructions to provision a resource. A **group owner** or
    **group admin** can create blueprints and make them visible to their group.

**group owner**
    is an individual who has the rights to create groups. each
    group must have an owner. In academic projects the group owner is
    typically the professor or the principal investigator. A group owner
    has accountability for the resources that a group uses. Group owners can
    also do everything a **group manager** does.

**group manager**
    is an individual who can create and enable and disable
    blueprints to regular group members. They are expected to be teaching or
    research assistans, junior researchers etc.

**blueprint templates**
    are created by the **admins**. A blueprint is created using a blueprint
    template.

**admins**
    are people who maintain the Pebbles installation. The much more
    comprehensive `admin documentation <http://cscfi.github.io/pebbles/>`_
    covers things admins should know about

**drivers**
    are the integrations Pebbles has with different resource providing
    systems, such as OpenStack, OpenShift or our custom DockerDriver

**ephemeral blueprints**
    some resources are ephemeral, i.e. all data on disk
    disappears when the resource expires

**persistent blueprints**
    instances tagged with peristent are the opposite of ephemeral:
    the disk is persisted in one way or another for at least some time.

Typical End-User Use Case
=========================

#. User logs in to Pebbles using SSO
#. User is given a **join code** for a group
#. User pastes/types the **join code** into Pebbles
#. User locates a Blueprint (called a resource in the user UI because they
   don't need to know what blueprints are)
#. User starts a blueprint
#. User uses the resources provided by the blueprint


Creating Blueprints
===================

First a Pebbles **admin** has to create a blueprint template. How to do so
is not covered here but you can contact your admins to make requests.

#. Go to Blueprints tab
#. Select the BlueprintTemplate which you wish to use and click Create
   Blueprint
#. **select the for which the blueprint will be made available**
#. Fill in other variables
#. Click Create
#. Activate the blueprint by clicking **Activate**

The configurable variables depend on what fields the **admins** have made
available for editing. Environment variables are available to the process
started inside the container so they can be used to configure the processes
in case the software supports

Bootstrapping environments
==========================

Most of the time it is desirable to have some pre-downloaded data or code
files present on the system. For a university course you might download the
templates.

The builds/scripts directory of
`notebook-images <https://github.com/CSCfi/notebook-images/>`_ contains
the scripts run at the start of containers. The bootstrap/ directory
contains examples of how to use the files. If you have multiple files and/or
repositories then it is suggested that you download a shell script and
execute it in the background. The script can be downloaded from anywhere as
long as it is publicly available on the Internet.

Note:

- if you have a persistent volume then if you blindly download things you
  **may override any changes a user made** on the disk. Write your shell
  scripts accordingly.
- the peristent disk is shared by all Blueprints on the same back-end so you
  should have your data inside a subdirectory
- most blueprints are containers which run as a pseudorandom UID for the
user
- both Jupyter Notebooks and RStudio support running an HTML-based shell so
  you can try out your commands before

The suggested workflow is:

#. boot a **clean** blueprint with no startup script
#. using the editing features in Jupyter notebooks / RStudio server edit a
   shell script file
#. **debug** running the shell script in the clean blueprint
#. when you are satisfied upload the file somewhere and set the system up to
   download and run it automatically


Environment variables
=====================

The containers built from `notebook-images
<https://github.com/CSCfi/notebook-images/>`_ all understand common
environment variables

AUTODOWNLOAD_URL
    A URL to be downloaded as a file to the local system, e.g
    "http://example.org/example.sh"
AUTODOWNLOAD_URL_FILENAME
    A target name for the URL, e.g. "data.zip"

The Jupyter notebooks containers additionally have

AUTODOWNLOAD_EXEC
    The name of a file to execute **before** starting Jupyter notebooks. If the
    script is essential for starting the notebook(s) then it should be
    run with AUTODOWNLOAD_EXEC.
AUTODOWNLOAD_EXEC_BG
    The name of a file to execute **in the background** just before starting
    Jupyter notebooks. If your script e.g. loads data over a slow network
    connection and that data isn't needed for the first 15 minutes,
    AUOTODOWNLOAD_EXEC_BG is a good alternative.

.. code-block:: shell

   AUTODOWNLOAD_URL=https://example.com/~user/scripts/prepare.sh
   AUTODOWNLOAD_EXEC=prepare.sh

Or a more complex example that forces the name of the downloaded artefact
and runs it in the background.

.. code-block:: shell
   AUTODOWNLOAD_URL=https://example.com/~user/scripts/nonsensical_name.sh
   AUTODOWNLOAD_FILENAME=bootstrap_slow.sh
   AUTODOWNLOAD_EXEC_BG=bootstrap_slow.sh

Git
---

Git wants to always know who the user is. Specifically it needs a username
and an email address. The UID in containers is a pseudorandom number and not
 available in the local /etc/passwd, which confuses git.

The workaround is to set

GIT_COMMITTER_NAME
    A human-readable name.

GIT_COMMITTER_EMAIL
    Something that looks like a valid email address.

Using HTTPS urls is preferred over using SSH for GitHub/GitLab. Note that if
the user has two-factor authentication enabled in GitHub they will need a
token to use the HTTPS interface.

Root Privileges
---------------

Most blueprints won't run the bootup script with root privileges for
security reasons. In some rare cases this cannot be avoided. In those cases
the user should be able to use `sudo` to become root.

**It is recommended to do all configurations and installation as the user
whenever possible.**