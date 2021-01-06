
# NovemberFuchsia [concept, not implemented yet]

this project hasn't been created yet - these are the concepts of the project I'm working on

the goal for this project is to make database creation and management easier, and to integrate that easily with flask

this is a flask-first application

# concepts

## adapter support

- currently supports **flask** and **firestore** only, but feel free to contribute for other databases too, I do intend to support them later on

## tiny example

```py
import firestore
import november_fuchsia as nf

from flask import Flask, request

# define all your database tables here
# or in a seperate .novemberfuchsia file (read more about it below)

dbs = nf.Abstract("""
User || the users table
-----------------------------------------
email         :  id  | str | emailCheck           || the email of the user
first_name    :        str | lenlim(1, 100)       || the first name of the user
last_name     :        str | lenlim(0, 100)       || the last name of the user
phone_number  :        str | phoneCheck  | unique || the phone number of the user
age           :        int | numlim(18, 123)      || the age of the user
""")


dbs.connect("google-firestore", firestore.Client())

# you may now subclass both classes to add additional functionality to them
Users = dbs.tables.get_db("User")
users = Users("users-v1.0.0")

app = Flask(__name__)
dbs.init_app(app)

@app.route("/user/create")
def create_user():
    user = users.User(
       email        = requests.args["email_id"],
       first_name   = requests.args["first_name"],
       last_name    = requests.args["last_name"],
       phone_number = requests.args["phone_number"],
       age          = requests.args["age"]
    )
    return f"created user {user.firstname} !"

@app.route("/user/get")
def get_user():
    email = requests.args["email"]
    user = users.get(email)
    if user:
        return f"got user {email}, firstname : {user.firstname}, lastname : {user.lastname}"
    else:
        return "user does not exist"

if __name__ == "__main__":
    app.run()
```

## lifecycles, caching and class-database-connect

so one of the core concepts here are the lifecycles

- database connections are created ONLY when a db object is newly asked for 
- database writes are done only in the end in bulk (though you may commit early via dbs.commit_early())

## Complex example

```
User || the users table
---------------------------------------------------------------------------------
first_name        :      str | lenlim(1, 100)
last_name         :      str | lenlim(0, 100)
email             : id | str | emailCheck  | unique
phone_number      :      str | phoneCheck        || the phone number of the user
age               :      int | numlim(18, 123)   || the age of the user

joined_on         : datetime | rangelimit(12/12/2001, today) 
                             || joining date of the user
                             
account_withheld  :     bool | default(false)
                             || if the users account has been withheld
                             || or closed by the company for some reason
                             || NOTE : this prevents the user from creating 
                             || any additional accounts
                             
address           : relation | connected(Address via Address.id) 
                             | notRequired
                             | reverseConnect
                             || connected via the id of the row
                             
Address
---------------------------------------------------------------------------
id           : id | auto generated uuid4
               || if no id is specified, this exact line above
               || will be automatically
               || added to the table
add_line_1   : str | lenlim(1, 200)
add_line_2   : str | lenlim(1, 500) | notRequired
country      : str | lenlim(1, 100)
state        : str | lenlim(1, 100)
city         : str | lenlim(1, 100)
pin_zip_code : str | lenlim(1, 10)
```
put the above lines in a file called `databases.novemberfuchsia`
entities are by default `required` so you need to add the `notRequired` tag
in case you don't want something

you can then get this file via code via
```py
import november_fuchsia as nf
dbs = nf.Abstract(fromfile="databases.novemberfuchsia")
```

you may also create multiple files and get all tables
```py
import november_fuchsia as nf
dbs = nf.Abstract(fromfiles=["user.novemberfuchsia", "address.novemberfuchsia"])
```

now you should see some formatting changes if you run the above lines of code, in the th

then retrieve the databases via
```py
UsersBase = dbs.tables.get_db("User")
Addresses = dbs.tables.get_db("Address")

# user affiliated actions
@UsersBase.override_base
class User(UsersBase.User):
    @property
    def fullname(self) -> str:
        """get the fullname of the user"""
        return self.first_name + self.last_name

# user collection affiliated actions
class Users(UsersBase):
    def get_users_in_age(
            min_age : int, 
            max_age : int) -> list[User]:
        """get users within an age range"""
        # get just the way we normally do in firestore
        return self.collection.
        
        
users = Users("users-v0.0.1") # the name of the collection to store in, in firestore

```

## The .novemberfuchsia file

- this file is auto-formatted to ensure consistency across files
- this means that all the syntax will be formatted into a style that the code authors liking
- auto-formatting doesn't add any logical changes, it only adds changes for consistency and formatting
