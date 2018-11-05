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

2. Adding the required codes as stated in official document [Custom OAuth2 configuration](https://superset.incubator.apache.org/installation.html#custom-oauth2-configuration)

3. Since the OAuth2 of *OriginSSO* doesn't support HTTPS, `flask-builder` will complains about this stating that it is not secure. To temporary allow this to happen, in the `superset_config.py`, add in the following code
```python
import os
os.environ['OAUTHLIB_INSECURE_TRANSPORT'] = '1'
```

4. Remember to tell virtualenv to read up the new path for our setting by adding the following line into 
`export PYTHONPATH="/home/superset/superset_setting` where `superset_setting` should contains the setting files you created after going through step 2 (`superset_config.py` and `custom_sso_security_manager.py`)

 


# 4. Unsolved issues
1. How do we enable two way of authentication? This issue is mainly the problem wit
<!--stackedit_data:
eyJoaXN0b3J5IjpbLTE3OTA2ODYxNzUsMzk2MjUwOTk5LDE4Nj
E4NzcxNjksLTE2NTE2NzMyNDIsMTUwODY0ODU5NywtMTMxMDkx
MDI1MywxODU0NzkwNjU4LC05MDA0MDA0NzQsLTIzNjk4OTg5NS
wyMTE2ODE3NDQ4LC05MDgyNTM1MjJdfQ==
-->