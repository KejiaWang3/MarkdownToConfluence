## Environment Variables

- For getting user credentials, consider using environment variables.
- This makes it so users of the script don't have to go in and edit the script, and change the username and password. Instead, they can just save their username/password as an environment variable and then run the script.
- This is also more secure, as you do not have to place your username/password in the script.
- The downside is you have to remember to save your credentials as an environment variable before you run the script.

Sample implementation:

```sh
# Before running the python script, user saves their username, password as an environment variable:
export JIRA_USER="kejia.wang@gov.bc.ca"
export JIRA_PASSWORD="ATATT3xFfGF0_keIaiyX7wd..."
```

```py
# Inside addGroup.py, we first need to import os library to read the environment variable:
import os

# We can now access the environment variables we saved previously, like this:
JIRA_USER = os.environ.get("JIRA_USER")
JIRA_PASSWORD = os.environ.get("JIRA_PASSWORD")

# If we printed, we could see our username and password:
print(JIRA_USER) # Prints: kejia.wang@gov.bc.ca
print(JIRA_PASSWORD) # Prints: ATATT3xFfGF0_keIaiyX7wd...

# We can now use these credentials to initialise Jira:
jira = Jira(
    url='https://aubs-sandbox.atlassian.net',
    username=JIRA_USER,
    password=JIRA_PASSWORD
)
```


## Dry Run

- It sounds like Aubrey wants to implement `dry-run` functionality
- This basically allows us to run the script without actually applying anything to Jira, and allows us to print to console, generate report, etc.
- This might looking something like this:

- Say we want to do a dry-run of our script. When we run the script, we include the `--dry-run` flag:

```sh
python3 addGroup.py --dry-run
```

- We can now capture this within our script, and modify script behavior if this flag is include. For example:

```py
import sys
from atlassian import Jira

DRY_RUN = '--dry-run' in sys.argv # Check to see if --dry-run flag is an argument when we ran the script
print(DRY_RUN) # prints: True

# Dummy user data:
user_info = {
  'name': 'John',
  'group': 'admin'
}

log = []

# We can now control behavior based on DRY_RUN value:
if not DRY_RUN:
  jira.add_user_to_group(user_info)
else:
  # If dry-run enabled, just print to console
  log.append(f'Adding user {user["name"]} to group {user["group"]} ...')

# write log to a log file
```

- This implementation is fine for simple dry-run functionality, but if the script has a bunch of `if not DRY_RUN: ... else: ...` blocks it might get a bit more cumbersome.
- For more advanced dry-run functionality, consider using the `dryable` library: https://github.com/haarcuba/dryable
- Basically it allows you to add a decorator to your function calls and controls how that function executes while dry running. For example:

```py
import dryable
from atlassian import Jira


@dryable.Dryable( logging_msg = 'Would have called {function} with args {args} (--dry-run)' )
def createNewUser(user):
  jira.create_new_user(user)

@dryable.Dryable( logging_msg = 'Would have called {function} with args {args} (--dry-run)' )
def addUserToGroup(user):
  jira.add_user_to_group(user)

users = ["a", "list of", "users" ]

for user in users:
  createNewUser(user)
  addUserToGroup(user)
```


-----


## Using Virtual Environments

- A virtual environment (or a venv) is a way for developers to create a separate python environment to run a given script.
- The primary reason why we want to use a venv is to ensure each person running the script is using the same version of a given package.
- For example, let's say you have `Jira 1.0.0`, on your local computer. You are working with another developer, and they `Jira 3.0.0` on their machine. They complete some work, and they hand over the script to you. You run the script, and you find it doesn't work. You do some searching and you find the reason why: the script is calling a function that is available in `Jira 3.0.0` but is not available in `Jira 1.0.0`.
- Virtual environments are used to solve this problem: they ensure each developer is using the same version of the dependencies.

Let's walk through an example of how these work:

- First, we need to create a virtual environment. This is typically done within the root directory of our project. To do this, run the following:

```sh
# Run: python3 -m venv <name-of-virtual-env>
# We are calling the venv module inside python (ie: python3 -m venv) and then specifying the name of the virtual environment. Typically, venvs are named venv. So to create a venv with that name, we run:

python3 -m venv venv
```

- Once are venv is created, we need to activate it. This is achieved by running:

```sh
# On mac/linux:
source venv/bin/activate

# On windows:
.\venv\Scripts\activate
```

- With the venv activated, we should see the name of the venv within our command line. For example, our command line might look something like:

```sh
(venv) ~/Add-Group $ 
```

- Now that our venv is activated we can start install dependencies. In our case, we have a `requirements.txt` file that outlines our dependencies, so we should install our those within our venv. To do that we can run:

```sh
pip3 install -r requirements.txt
```

- Something that's important to note: this installs the dependencies ONLY within our venv. It does NOT install them globally, on our local machine.
- If you find that you run your script, and you see something like `module not found`, you likely need to activate the virtual environment, and perhaps install the dependencies.
- Once the dependencies are installed, we can run our script like usual.
- If you need to deactivate the virtual environment, you can do so by simply running:

```sh
deactivate
```

- It took me some time to understand the reason for using virtual environments, and in my experience, it's generally a bit of a weird concept for those unfamiliar with it to understand!
- If you come from a JavaScript background, they are basically the same as `node_modules`, `package.json`, etc.
- Using them is a best practice for Python development. Even if it's not fully clear why they are used, they should still be used within your Python projects.
- For more reading, check out: https://realpython.com/python-virtual-environments-a-primer


## Pandas

- Pandas is a wonderful, powerful tool. However, at this script's current stage of implementation, I am wondering if it is necessary.
- If there is some functionality for pandas planned for the future of this script, disregard this section!
- From what I can see, it seems pandas is mainly used to read in a csv, iterate through the rows, and apply some functionality to each row. 
- There are libraries built in to python we can leverage for this functionality. For example, the `csv` library.
- Here is sample implementation of csv:

```py
# First, import csv. No need to install as it is built in to Python.
import csv

with open('user_information.csv', newline='') as csvfile:
    reader = csv.DictReader(csvfile)
    for row in reader:
        # Each row in the csv is converted into a dictionary:
        print(row) # prints: {'Users': 'AubreyRobertson', 'UserId': ...}

        # Values in row are accessible via key-value pairs:
        print(row['Users']) # prints: 'AubreyRobertson'
        print(row['UserId']) # prints: '557058:6193f5d3 ...'
```

- A few reasons why we might want to avoid pandas, if possible:
  - Developer knowledge
    - Pandas requires some knowledge to use and it has some quirks. If the developer does not have experience with pandas, it will take them longer to understand how to use the script.
  - Size
    - If we can, we should aim to keep our scripts as lightweight as possible by minimizing unnecessary dependency usage


## Script Entrypoint

- Typically within our script, we want to include an `if __name__ == '__main__':` block at the entrypoint of our script.
- This is another one of those weird quirks of python, but it is good practice to include within your scripts
- Essentially, it prevents code from running when you import a module into your script

- For example, let's say this script gets really big, and we want to start breaking it up into multiple files.
- We have our main script - `addGroup.py` - and we create another function to contain some utilities, which go in `utils.py` file.

- Here is what the contents of our files look like:

```py
# Inside addGroup.py

def main():
  print("Running main function!")

main()
```

```py
# inside utils.py

def some_utility():
  print("Performing some utility!")

some_utility()
```

- Say we want to import utils.py into addGroup.py, and use the some_utility function within addGroup.py
- So we make the following changes:

```py
# Inside addGroup.py

import utils

def main():
  print("Running main function!")

main()
utils.some_utility()
```

- When we run the script, we see something that might be surprising: `some_utility()` is actually called twice! The output will look something like:

```
Performing some utility!
Running main function!
Performing some utility!
```

- What's happening here is when `utils.py` is imported, all of the code within that script is run first, including the call to `some_utility()`.
- Then, after the import, the rest of the code inside `addGroup.py` is run: the call to `main()` runs, followed by `utils.some_utility()`
- So, to prevent the `some_utils()` function from being called, we can guard it using an `if __name__ == '__main__':` statement. So we change out code to the following:

```py
# Inside utils.py
def some_utility():
  print("Performing some utility!")

if __name__ == '__main__':
  some_utility()
```

```py
# inside addGroup.py

import utils

def main():
  print("Running main function!")

if __name__ == '__main__':
  main()
  utils.some_utility()
```

- Now when we run the script, we say the expected output: `main()` runs first, followed by `utils.some_utility()`

```
Running main function!
Performing some utility!
```

- It's important to note, during the import, all of the code within utils.py is still run... we are just preventing the `some_utility()` function from being called by placing it in the `if __name__ == '__main__':` block.

- So, what this means for the script:
- Suggest adding the following to `addGroup.py`

```py
# Changing line 68 to:
if __name__ == '__main__':
  read_csv()
```

- And then if the scripts gets broken up into multiple files/modules, include `if __name__ == '__main__':` blocks inside of those as well.


## Bare Exceptions

- Ideally want to avoid using bare exceptions (that is, an `except` without specifying an exception type)
- Reason: this catches and handles ALL exceptions. This includes exceptions like `KeyboardInterrupt`.
  - Basically, if there is an issue with the script and the user wants to stop it from running, they will not be able to stop the script using ctrl + c... the only way to exit will be to close the program
- At the bare minimum, we want to include `except Exception`, which will allow KeyboardInterrupt and permit the user to quit by pressing ctrl + c
- Ideally, want to use more specific exceptions, then broaden to handle more generic exceptions
- Eg:

```py
try:
  jira.add_user_to_group( ... )
except JIRAError:
  # It seems Jira only generates one type of exception, so we handle that here:
  print('User exists in this group')
  # If we had logging set up we might include something like:
  # logger.info(e)
except Exception as e:
  # If a more generic error occurs, we handle that here:
  print('An unexpected error occurred.')
  # If we had logging set up we might include something like:
  # logger.error(e)
```