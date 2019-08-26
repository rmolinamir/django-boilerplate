# Django REST API

Django REST API concepts.

---

## Vagrant

### Initialization

To initialize vagrant, run:

```commandline
vagrant init ubuntu/bionic64
```

Where `ubuntu/bionic64` is the type (OS) of the created server, there are more options available in the vagrant website.

### Basics

- The box type is `ubunty/bionic64`.
- To install the virtual machine run: `vagrant up`.
- To destroy the virtual machine run: `vagrant destroy`.
- To start the virtual machine run: `vagrant ssh`.
- While inside the virtual machine, the directory that is synchronized with the root folder of this project is the `vagrant` folder.
- While inside the virtual machine, the directory of the synchronized folder will look like `vagrant@ubuntu-bionic:/vagrant$` in the terminal.

### Activation *(inside the virtual machine)*

**1. Installation:**

We need to create a Python virtual environment before creating a Django App. To create and activate a Django environment, run:

```commandline
python -m venv ~/env
```

Note that the directory of this installation is outside the `vagrant` folder, this is to avoid the installed dependencies being synchronized with the root folder.

**2. Activation:**

An active virtual environment is needed so that all of the installed dependencies are pulled from the virtual environment instead of the base operating system. To activate the local environment, run:

```commandline
source ~/env/bin/activate
```

So long as python virtual environment was installed in `~/env`, this command line will run correctly. You will know that you're working on a virtual environment because the name of the virtual environment that is being worked on will appear in brackets as a prefix to the command line input, such as `(env) vagrant@ubuntu-bionic:/vagrant$`.

To deactivate the virtual environment (obviously while it is active) run:

```commandline
deactivate
```

- Useful Python virtual environment cheatsheet: https://python-guide.readthedocs.io/en/latest/dev/virtualenvs/

---

## Dependencies

The required Python packages are located in the `requirements.txt` file as a best practice. This file will include specific packages and versions (pinning them down to specific versions is intended).

For more information about the packages, go to https://pypi.org/.

To install the required dependencies, run inside the virtual machine (while activated):

```commandline
pip install -r requirements.txt
```

---

## Django

**1. Create a new Django project:**

**While in a git bash command line inside the vagrant machine terminal, and while it is active**, run:

```commandline
django-admin.py startproject [project_name] .
```

What this does is that it will run the `django-admin.py` script that will start a new project by running `startproject` and set a name of `[project_name]` in the specified location `.` (root folder). If a location is not specified, it will create a new sub-folder.

**2. Create a new Django app (within our project):**

A Django project can consist of one or more sub applications within a project that can consist of different functionalities within a project.

To create a sub-application within a project, run (*inside the vagrant machine terminal sync folder, and while it is active*):

```commandline
python manage.py startapp [project_app_name]
```

Where `startapp` will create a new app, and `[project_app_name]` is used to set the name (and will appear as a new sub-folder).

**3. Enable an app within our project**:

To install/add new apps to the project, open the `settings.py` file inside the project folder, and add the app's name to the `INSTALLED_APPS` list.

For example:

```python
INSTALLED_APPS = [
    'django.contrib.admin',
    'django.contrib.auth',
    'django.contrib.contenttypes',
    'django.contrib.sessions',
    'django.contrib.messages',
    'django.contrib.staticfiles',
    'rest_framework',  # Django REST framework.
    'rest_framework.authtoken',  # Allows usage of authentication tokens out of the box with the REST framework.
    '[project_app_name]', # Our Django created app.
]
```

**4. Running the Django development server**:

While connected to the vagrant box, working on an environment (`(env)`), run:

```commandline
python manage.py runserver 0.0.0.0:8000
```

This command will start development server available on all network adapters in port 8000 (as set in the `VagrantFile`).

To stop the server, type `CTRL+C`.


--- 

## Django Components

From the components *closest* to the **database tables**, to the components that are *closest* to the **HTTP requests that receive JSON data**:

- **Models**: These describe how we want the structure of our data so that Django can translate Python objects to tables in a database.

- **Serializers**: These translate the data from the HTTP request into a Python object.

- **Views**: These receive the HTTP requests, passes it to a **Serializer** to translate the data from the HTTP request into a Python object, that can then in turn be passed to the **model**, and translated to a row in the database.

For example, the `perform_create()` function in a `ModelViewSet` is the entry point in our hypothetical application. So when we want to create a new user, we make a HTTP POST request which gets received by `perform_create()`, which passes it through the Serializer to convert the raw JSON to a Python object and then passes it to the model to save it in the database.

---

## Deployment to AWS

- *Optional pre-requirement*: Add a SSH Key Pair to AWS to use SSH authentication (optional but recommended - **the SSH Key Pair is for the user and not the actual Django server**).

To do this, go to the AWS website, click on the *Services* tab, go to the *EC2* section, then go to the *Network & Security* sub-section, then click on *Key Pairs*, and finally follow the instructions to add the SSH Key Pair (it's recommended to give it a name that matches the laptop or computer being used).

#### Deployment Process:

1. Create and launch an EC2 instance.
    - It is important to choose an appropriate Amazon Machine Image (AMI).
    - Configure the instance details, and allow HTTP access inside the "Configure Security Group" section by adding a new rule.
2. Create deploy scripts, take a look at the `deploy` folder in the root directory.
3. Configure the `settings.py` file, particularly the `DEBUG`, `STATIC_URL`, and `STATIC_ROOT` variables.
4. Make the deploy scripts executable in the AMI by running `chmod +x deploy/*.sh`
5. Connect to the launched EC2 instance.
    - 5.1. Copy the Public DNS (IPV4) address in the dashboard, then connect to it in a bash terminal in the root directory of the project by running:
    ```commandline
    ssh ubuntu@ec2-54-242-255-112.compute-1.amazonaws.com
    ```
   Note: you might be prompted for security SSH authentication, follow the instructions. If you get stuck, start by giving [this thread a read](https://superuser.com/questions/988185/how-to-avoid-being-asked-enter-passphrase-for-key-when-im-doing-ssh-operatio).
   - 5.2. Download and run the script that was created to connect to the server (`deploy/setup.sh`. To do this, pull the script from GitHub into the server, then run the script to setup the server for deployment **(replace the URL in the command with the one you will be using)**. To do this, run in the following command line in a bash terminal inside the root directory (while connected to the previous AMI's ubuntu machine):
   ```commandline
   curl -sL https://raw.githubusercontent.com/rmolinamir/django-rest-api/master/deploy/setup.sh | sudo bash -
   ```
6. Update the `ALLOWED_HOSTS` list inside `settings.py` to be able to remotely access the Django Admin Browsable API. Add the Public DNS (IPV4) address and the localhost (`'127.0.0.1'`).
7. Run the update script on the server, by running the `deploy/update.sh` script (You have to move into the project's root directory, you can do this by running `cd ../../usr/local/apps/rest-api` in Windows 10). **Always do this to implement any change.**
