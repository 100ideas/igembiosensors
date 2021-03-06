iGEM Biosensors DB
==================

The iGEM Biosensors DB (now known as "SensiGEM" and hosed publicly at
[www.sensigem.org](http://www.sensigem.org/)) catalogues over 220 biosensors
developed by iGEM teams, including sensor inputs, outputs, applications,
category, and awards earned. Both the source code and biosensor data are made
publicly available under the MIT license, meaning that in addition to using
[www.sensigem.org](http://www.sensigem.org/), teams may host their own versions
of the database.

Installation
------------

1. Create Python 2 virtual environment for project.

        mkdir ~/.virtualenvs
        cd ~/.virtualenvs
        virtualenv2 igembiosensors
        source igembiosensors/bin/activate
        cd -

2. Install Django 1.6. As this version is still in beta as of this writing, you must install from GitHub.

        pip install git+git://github.com/django/django.git@1.6b4

3. Install dependencies.

        pip install django-taggit django-widget-tweaks

4. Clone and configure project.

        git clone https://github.com/jwintersinger/igembiosensors.git
        cd igembiosensors/igembiosensors
        cp settings_deployment.py.example settings_deployment.py
        # If you please, change database. PostgreSQL works well in production. If
        # you're not using SQLite, see
        # https://docs.djangoproject.com/en/dev/ref/settings/#databases for
        # instructions on configuring DB settings.
        vim settings_deployment.py
        cd ..
        
5. Synchronize database.
        
        # When asked if you wish to create a superuser account, select "yes", then
        # enter a username and password. (You need not enter an e-mail address.) If
        # you forget your password or need to create another superuser account
        # later, see these two commands:
        #  python2 manage.py help changepassword
        #  python2 manage.py help createsuperuser
        python2 manage.py syncdb

6. Import initial data set. Skip this step to start with an empty database.

        python2 manage.py loaddata backups/biosensors_data.json
        
7. Run server.
        
        python2 manage.py runserver 0.0.0.0:8000
        # Now, access http://<your_ip>:8000 in your web browser.
        
8. Set up additional user accounts. Access `http://<your_ip>:8000/admin/` in your web browser. You have two options:
    1. Make additional users superusers. This means that these users will be able to add new users themselves.
        1. Click the Add link next to Users under the Auth heading.
        2. Enter a username, password, and password confirmation, then click Save.
        3. On the next screen, under the Permissions heading, check both the `Staff status` and `Superuser status` checkboxes.
        4. Click Save.

    2. Make additional users limited users. They will not be able to add new users themselves.
        1. Create a group setting permissions for these users. You need do this only once.
            1. Click the Add link next to Groups under the Auth heading.
            2. Enter an appropriate group name, such as `Project editors`.
            3. Under `Available permissions`, in the text field next to the
               magnifying glass, type `biosensorsdb`, then click `Choose all`.
               Note that you  can, for example, forbid users from creating new
               categories -- simply omit the `Can {add,change,delete} category`
               permissions. Users will be able to add projects to existing
               categories, but not create or modify the list of categories.
            4. Click Save.

        2. Add the users.
            1. From the administration home, Click the Add link next to Users under the Auth heading.
            2. Enter a username, password, and password confirmation, then click Save.
            3. On the next screen, under the Permissions heading, check `Staff status` checkbox.
            4. Under `Available groups`, select the group you created in the
               previous step, then click the right arrow icon to add the user to
               this group.
            5. Click Save.

9. After entering production data, backup the database.

        # Use --natural flag because django-taggit references contenttypes.
        # Note that user accounts are *not* exported, thereby allowing you to
        # share your data without exposing user credentials.
        #
        # Note that, until the next release of django-taggit, one must fix a
        # bug in the package before dumpdata will work, as per
        # https://github.com/alex/django-taggit/issues/155#issuecomment-24561900.
        python2 manage.py dumpdata --indent=2 --natural biosensorsdb taggit > backups/biosensors_data.json

        # Alternatively, to backup the database as a whole (including user
        # accounts -- this means you must be cautious sharing your backup, as
        # it will include user credentials), run this command if you're using
        # PostgreSQL:
        pg_dump -U micand igembiosensors | bzip2 > backups/db.sql.bz2
