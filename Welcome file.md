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

5. In the official documentations, the custom `custom_sso_security_manager.py` didn't import the module `logging`

```python
from superset.security import SupersetSecurityManager
import logging

class CustomSsoSecurityManager(SupersetSecurityManager):

    def oauth_user_info(self, provider, response=None):
        logging.debug("Oauth2 provider: {0}.".format(provider))
        if provider == 'originsSSO':
            # As example, this line request a GET to base_url + '/' + userDetails with Bearer  Authentication,
            # and expects that authorization server checks the token, and response with user details
            me = self.appbuilder.sm.oauth_remotes[provider].get('userDetails').data
            logging.debug("user_data: {0}".format(me))
            return { 'name' : me['name'], 'email' : me['email'], 'id' : me['user_name'], 'username' : me['user_name'], 'first_name':'', 'last_name':''}
```
 

## Controlling Access Group
To control which OAuth2-authenticated user have Admin privilege, Alpha privilege, Gamma privilege, and etc. In the `CustomSsoSecurityManager` class, overwrite the function `auth_user_oauth`. For our use case, we would like to check if the user's belong to either *'SA'* or *'SSA'*, if yes they should be granted Admin access. 

**ORIGINAL**

```python
def auth_user_oauth(self, userinfo):
        logging.debug('Using custom "auth_user_oauth".')
        """
            OAuth user Authentication

            :userinfo: dict with user information the keys have the same name
            as User model columns.
        """
        if 'username' in userinfo:
            user = self.find_user(username=userinfo['username'])
        elif 'email' in userinfo:
            user = self.find_user(email=userinfo['email'])
        else:
            log.error('User info does not have username or email {0}'.format(userinfo))
            return None
        # User is disabled
        if user and not user.is_active:
            log.info(LOGMSG_WAR_SEC_LOGIN_FAILED.format(userinfo))
            return None
        # If user does not exist on the DB and not self user registration, go away
        if not user and not self.auth_user_registration:
            return None
        # User does not exist, create one if self registration.
        if not user:
            user = self.add_user(
                    username=userinfo['username'],
                    first_name=userinfo['first_name'],
                    last_name=userinfo['last_name'],
                    email=userinfo['email'],
                    role=self.find_role(self.auth_user_registration_role)
                )
            if not user:
                log.error("Error creating a new OAuth user %s" % userinfo['username'])
                return None
        self.update_user_auth_stat(user)
        return user
``` 

**MODIFIED**

```python
def auth_user_oauth(self, userinfo):
        logging.debug('Using custom "auth_user_oauth".')
        """
            OAuth user Authentication

            :userinfo: dict with user information the keys have the same name
            as User model columns.
        """
        if 'username' in userinfo:
            user = self.find_user(username=userinfo['username'])
        elif 'email' in userinfo:
            user = self.find_user(email=userinfo['email'])
        else:
            log.error('User info does not have username or email {0}'.format(userinfo))
            return None
        # User is disabled
        if user and not user.is_active:
            log.info(LOGMSG_WAR_SEC_LOGIN_FAILED.format(userinfo))
            return None
        # If user does not exist on the DB and not self user registration, go away
        if not user and not self.auth_user_registration:
            return None
        # User does not exist, create one if self registration.
        if not user:
            if len(set(userinfo['groups']) & set(('SA', 'SSA'))) > 0:
                logging.debug('Admin user detected.')
                user = self.add_user(
                    username=userinfo['username'],
                    first_name=userinfo['first_name'],
                    last_name=userinfo['last_name'],
                    email=userinfo['email'],
                    role=self.find_role('Admin')
                )
            else:
                user = self.add_user(
                        username=userinfo['username'],
                        first_name=userinfo['first_name'],
                        last_name=userinfo['last_name'],
                        email=userinfo['email'],
                        role=self.find_role(self.auth_user_registration_role)
                    )
            if not user:
                log.error("Error creating a new OAuth user %s" % userinfo['username'])
                return None
        self.update_user_auth_stat(user)
        return user
``` 

Notice how in the flow, we first check if the user's [`groups`] contains either the element in `('SA', 'SSA')` or not. If yes, we specify the `role` as `'Admin'`.

## Default OAuth2 provider 
It is intended in this use case, by default to select `originsSSO` as the OAuth2 provider. This can be achieved by taking the following steps:
1. Firstly, head to `.virtualenvs/superset/lib/site-packages/flask_appbuilder/templates/appbuilder/general/security`, inside of which you can locate the file `login_oauth.html`. Copy the file.
 
2. Go to the directory `superset` has been installed. (in this case, it will be in `.virtualenvs/superset/lib/site-packages/superset`.)
3. In this folder, go to the `templates/appbuilder/general` folder. 
4. Create a folder named `security`
5. Inside of this `security` folder, paste the copied `login_oauth.html` file
6. Edit this `login_oauth.html` file so that the following function is called right after the page is loaded.
	```javascript
	set_openid(url='/login/originsSSO', pr='originsSSO'); 
	```

## Custom icon for OAuth2
The problem with custom icon for OAuth2 is that `Flask` framework relies on **Font Awesome** type of icon when adding view (through the function `add_view()`). It is difficult to hijack this process as superset is build on top of the flask framework and that function is buried too deep to be able to easily overwrite.

## Redirecting logout superset to logout screen on Origin (AquilaOne)
All the processes and methods invoked in login page, authentication page, and log out page are handled by `class AuthView(BaseView)` which is defined in [flask_appbuilder/security/view.py](https://github.com/dpgaspar/Flask-AppBuilder/blob/master/flask_appbuilder/security/views.py)

From the source code, it can be observed that in the class, they defined **login** and **logout** process. What we want to achieve here is to overwrite the **logout** process so that they redirect user to AquilaOne logout page (http://poseidon:8896/sec/logout) instead of the superset index page. 

To do so, we can make use of the fact that superset consume [**Blueprints**](http://flask.pocoo.org/docs/1.0/blueprints/#blueprints) during initialization, and **blueprints** can be specified and passed to the initializer in the [config file](https://github.com/apache/incubator-superset/blob/master/superset/config.py#LC404).

The blueprint can be created by following this particular [tutorial](http://flask.pocoo.org/docs/1.0/tutorial/views/#logout). The goal here is to define our own custom **logout** function and then route the path ('/logout/') to our custom **logout** view. This can be specified by using the decorator.
```python
@app.route('/logout/')
```

After we define our own custom **logout** view with this new blueprint, we save the file under the name `auth.py` which shall then be placed in the same folder as the original config file `superset_config.py`. In the end our `auth.py` file should look like this:

```python
from flask_login import logout_user, login_user
from flask import Blueprint, redirect

bp = Blueprint('auth', __name__, url_prefix='')

@bp.route('/logout/')
def logout():
    logout_user() # We want to call the same method as called by original 
				    #implementation in order to not mess up things
    return redirect('http://poseidon:8896/sec/logout')
```

In the `superset_config.py`, we import the `blueprint` defined in `auth.py`, and then subsequently pass it into the parameters as the following:
```python
from auth import bp

BLUEPRINTS = [bp]
``` 

Now if everything is done correctly, pressing the `logout` button on superset website will invoke this custom logout procedure, which will then redirect us to origin's logout page.

## Removing "register" button on login page.
Do not attempt to set `AUTH_USER_REGISTRATION` as `False`. This will make user which has access to AquilaOne, but hasn't log in before unable to log in. Because when setting the parameter to `False`, we are essentially blocking 

# 4. Unsolved issues
1. How do we enable two way of authentication? This issue is mainly the problem with `flask-appbuilder` since it doesn't allow two `AUTH_TYPE` [flask-appbuilder base configuration (see AUTH_TYPE)](https://flask-appbuilder.readthedocs.io/en/latest/config.html)
2. At the OAuth2 provider side, admin needs to register the superset application's port number at AITSP so that user can be redirected after they have successfully identify themselves.
3. OAuth2 provider side is prone to internal server error, which is as shown in the document below:

<!--stackedit_data:
eyJoaXN0b3J5IjpbMTMzMTYxMDAzNSwtMTgwOTgyMjQyMCwtNz
AxODUxNTY2LC0xOTAyMDY2MTUzLDYxMjU0NjEzNywxNzE2ODE1
NTUzLDE1Mjk2OTYwODEsMzk2MjUwOTk5LDE4NjE4NzcxNjksLT
E2NTE2NzMyNDIsMTUwODY0ODU5NywtMTMxMDkxMDI1MywxODU0
NzkwNjU4LC05MDA0MDA0NzQsLTIzNjk4OTg5NSwyMTE2ODE3ND
Q4LC05MDgyNTM1MjJdfQ==
-->