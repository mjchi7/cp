# 1. Creating *virtualenv* using `virtualenvwrapper`

1. After logging into terminal, first type the following command
`pip freeze` 
and ensure that `virtualenvwrapper` is present in the list. 
2. To create a virtual environment, simply enter the following command to create a virtual environment named *superset*.
`mkvirtualenv superset` 
3. If installation is successful, you should notice the terminal will be preceded by parentheses and inside of which is the name given to the virtual environment. For example
`(superset) [superset@AITSP ~]$`

# 2. Install superset
While still in the virtual environment, do the following:

0. *This step is not documented in superset official installation guide*, but please ensure the following package is already installed in order for the following steps to work:

`sudo apt-get install libsasl2-dev` for Ubuntu
`sudo yum install cyrus-sasl-devel` for AWS EC2/EMR or **CentOS** 

For more information refers to this [link](https://stackoverflow.com/questions/48562383/sasl-saslwrapper-h2223-fatal-error-sasl-sasl-h-no-such-file-or-directory).

1. Firstly, as suggested by the official installation guide, always upgrade `pip` and `setuptools` to avoid any unforeseen circumstances.
`pip install --upgrade setuptools pip`

2. After preliminaries are done, do the following steps to install and initialize superset on the server (copied from the official installation guide)
```python
# Install superset
pip install superset

# Create an admin user (you will be prompted to set username, first and last name and
# finally, a password)
fabmanager create-admin --app superset

# Initialize the database
superset db upgrade

# Load some data to play with
superset load_examples

# Create default roles and permissions
superset init

# To start a development web server on port 8088. Use -p to bind to another port.
superset runserver -d
```

3. After installation, head to [http://localhost:8088](http://localhost:8088) and login using the credentials entered while creating the admin account. 

<!--stackedit_data:
eyJoaXN0b3J5IjpbMTg1NDc5MDY1OCwtOTAwNDAwNDc0LC0yMz
Y5ODk4OTUsMjExNjgxNzQ0OCwtOTA4MjUzNTIyXX0=
-->