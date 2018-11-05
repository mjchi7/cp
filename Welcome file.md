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

3. Installing superset
`pip install superset`

4. Create an admin user for superset. You will be prompted to set *username*, *first* and *last* name and finally, a *password*.
`fabmanager create-admin --app superset`

*Note: If the server complains about **unsupported UTF-8 locale issue**, simply enter the following command*
`export LC_ALL="en_US.UTF-8"` and try again. 
For more information about this issue, refers to this [link](https://askubuntu.com/questions/162391/how-do-i-fix-my-locale-issue).

5. Initialize the database
`superset db upgrade`

6. Load some sample data to play with
`superset load_examples`

7. Create default role and permission
`superset init`

8. To start a development web server on port 8088. Use -p to bind to another port.
`superset runserver -d`

9. After installation, head to [http://localhost:8088](http://localhost:8088) and login using the credentials entered while creating the admin account. 


# 3. Configuring OAuth2 

1. Install required modules:
`pip install flask-oauthlib`

2. Adding the required codes as in official documents

3. Remember to tell virtualenv to read up the new path for our setting
`export PYTHONPATH="/home/superset/superset_setting`

4. Since the OAuth2 of *OriginSSO* doesn't support HTTPS, `flask-builder` will complains about this stating that it is not secure. To temporary allow this to happen, in the `superset_config.py`, add in the following code
```python
import os
os.environ['OAUTHLIB_INSECURE_TRANSPORT'] = '1'
```

 



<!--stackedit_data:
eyJoaXN0b3J5IjpbMTg2MTg3NzE2OSwtMTY1MTY3MzI0MiwxNT
A4NjQ4NTk3LC0xMzEwOTEwMjUzLDE4NTQ3OTA2NTgsLTkwMDQw
MDQ3NCwtMjM2OTg5ODk1LDIxMTY4MTc0NDgsLTkwODI1MzUyMl
19
-->