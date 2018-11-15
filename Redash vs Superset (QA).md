## Question 1: Can we restrict access to certain datasource by user attributes?

### Superset

#### Short answer
Yes we can

#### Long answer
Say we want to have a bar chart that shows *staff name* v.s. *sales (RM)*. This bar chart, ideally should be a single bar chart created, and then the data it present will depends on who is currently viewing the chart. 

For instance, if a **state manager** is currently viewing the chart, he/she should be exposed of the data only in that particular state, so we want the same chart, to display the data for *staff name* and *sales (RM)* if and only if the *staff name* belongs to the said state. Moving on, if a **branch manager** is looking at the chart, the data being presented should only be *staff name* that are belongs to that particular branch.

If that's the cases you want, it can be done, but very elaborated. 

#### Approach 
In general, the idea is that we can add a conditioning filter at the data retrieved by SQL, so that the data retrieved depends on certain attributes of currently logged in user. 

For instance, in the vanilla build, we have the following attributes of a user:
> 1. username
> 2. fullname
> 3. email
> 4. userid
> 5. roles

What we can do is to *attach* a new type of attribute to all the user. For instance, **region**. So right now for every user, they will have the following attributes:
> 1. username
> 2. fullname
> 3. email
> 4. userid
> 5. roles
> 6. region <-- newly added.

With this new attribute, we can make use of the **JINJA TEMPLATE** built-in into the sql-lab, with which we can build some conditions into the SQL query so that the data retrieves depends on user's attribute. 

For instance, we can check if current user's region attribute is *Selangor*, the SQL query will modify itself (thanks to JINJA TEMPLATE) so that only *staff name* and *sales (RM)* belongs to *Selangor* will be retrieved, effectively having a dynamic visualization of the same chart.

#### Steps
##### 1. Installing dependencies
The required package doesn't comes with the original `requirements.txt`.
Do the following command
`pip install flask_babelpkg`

##### 2. Adding new user attributes
Follow [flask_appbuilder's step](https://flask-appbuilder.readthedocs.io/en/latest/security.html#extending-the-user-model) to extend User model and adding our new attributes. 

*notes: For OAuth, you need to overwrite the **useroauthmodelview** in your CustomSecurityManager*
```python
class CustomSsoSecurityManager(SupersetSecurityManager):
    user_model = MyUser # Your custom User model 
    useroauthmodelview = CustomUserOAuthModelView # Your custom UserModelView
```

*notes 2: if SQLAlchemy complains about \_\_table_args\_\_, when you are extending User, just re-assign the attribute `__name__` to whatever it is in the original implementation (usually it is `ab_user`) 

When both of that is complete, you need to override the function "add_user()" in your CustomSecurityManager class as well, so that the function can consume whatever new attributes you are adding.

##### 3. Modifying sqllite table to include the new attribute column
If you can afford to delete your db and start fresh, kindly go to `./.superset` and delete the `superset.db` inside of it.

##### 4. Modifying JINJA template
Go to `your_superset_installation_path/superset`, look for the file `jinja_context.py` add in a new function to retrieve your new attributes (refers to function like `current_username()` or `current_user_id()`

With that new function created (let's say in my case I have created `current_user_region()`, scroll down the codes and look for the class 

```python
class  BaseTemplateProcessor(object):
```
Register your newly created function in the list of `self.context`
For example

```python
self.context = {
	'url_param': url_param,
	'current_user_id': current_user_id,
	'current_username': current_username,
	'current_user_region': current_user_region,
	'filter_values': filter_values,
	'form_data': {},
}
```

Then in your SQL, you can access the function you registered with `BaseTemplateProcessor` as such

```sql
SELECT * 
FROM SOME_TABLE 
WHERE USER_REGION IN 
{% if current_user_region() == 'Selangor' %}
('Selangor')
{% else %}
('Perak', 'Perlis', 'Johor')
```

The following query will select the `WHERE` condition based on the returned value of `current_user_region`. If it returns `Selangor`, query will select data with `USER_REGION == Selangor`, or else it will select data with `USER_REGION == 'PERAK' or 'PERLIS' or 'JOHOR'`

For more information, refers [JINJA Page](http://jinja.pocoo.org/docs/2.10/templates/)

BUTTTT CAN USER CHANGE QUERIES THOUGH??????
<!--stackedit_data:
eyJoaXN0b3J5IjpbLTE2NDU1NDUzNjgsNDAyNTg1ODI3LDYyMz
IyNTYzOV19
-->