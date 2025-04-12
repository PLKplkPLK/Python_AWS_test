## Basic knowledge
1. `__main__.py` is a file in Python project's directory that is an entry point to the project.

2. When writing a module? You have to put the (not to be run) code inside `if __name__ == '__main__': ...` The code inside this if-statement will run only, when run like so: python3 module.py

3. `__init__(self, *args)`

4. Options:
- `'string' + str(variable)`
- `f'string {variable}'`
- `'string %d' %2`

5. I don't think that's possible. Conventions with `__var` and `_var`, are just conventions, it doesn't restrict access.

6. Function decorator.

7. `@staticmethod` receives only explicitly given arguments, while `@classmethod` receives class object as well. Static methods don't have to be created inside a class, but we might want to have them there, because it's logically correct (for it to be encapsulated there).

8. It automatically closes file at the end of execution.


## Python code fixing

- `if __name__ == "__main__"`

It just... it should be `"__main__"`, not `"main"`

- `self.age = age`

Instead of `self.age == age`, because that's not assigning, that's a check whether they're the same (and since that's a constructor, they won't be)

- `increase_count()` changed to 2 different methods: increase and get

I just thought it would be easier to use it like that - KISS

- `x = Person(p["first_name"], p["last_name"], p["age"])`

Arguments were just not in the correct order.

- print number of people at the end

Added a loop to iterate over threads and wait for them to finish with `threading.Thread.join()` method


## AWS Lambda
Permissions for Lambda functions:
- IAM
- Roles
- [func]-role-...
- inline policy


https://1rko02x0cc.execute-api.eu-north-1.amazonaws.com/kolo/user

with body:
```
{
    "first_name": "Everest",
    "age": 34
}
```

Lambda POST code:
```
import json
import boto3
import uuid


def insert_to_table(user) -> str:
    dynamodb = boto3.resource('dynamodb')
    users_table = dynamodb.Table('users')

    # how can we be sure it will be unique in db?
    guid = str(uuid.uuid4())
    user['user_id'] = guid
    users_table.put_item(Item = user)

    return guid

def lambda_handler(event, context):
    try:
        if (not 'first_name' in event) or (not 'age' in event):
            return {
                'statusCode': 400,
                'body': json.dumps("'first_name' and 'age' required.")
            }

        guid = insert_to_table(event)

        return {
            'statusCode': 200,
            'body': json.dumps({'GUID': guid})
        }
    except:
        return {
            'statusCode': 418,
            'body': json.dumps('Something\'s wrong :(')
        }

```


https://1rko02x0cc.execute-api.eu-north-1.amazonaws.com/kolo/user/{id}

Lambda GET code:
```
import json
import boto3


def read_from_table(guid: str) -> str:
    dynamodb = boto3.resource('dynamodb')
    users_table = dynamodb.Table('users')

    user = users_table.get_item(Key = {'user_id': guid})

    return user.get('Item')

def lambda_handler(event, context):
    try:
        user_guid = event.get('pathParameters').get('user_id')
        user = read_from_table(user_guid)

        if user is not None:
            return {
                'statusCode': 200,
                'body': json.dumps(user)
            }

        return {
            'statusCode': 404,
            'body': json.dumps('User not in db.')
        }
    except:
        return {
            'statusCode': 400,
            'body': json.dumps('Something went wrong. Sorry :(')
        }

```