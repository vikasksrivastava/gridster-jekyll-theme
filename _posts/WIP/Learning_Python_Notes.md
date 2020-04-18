
### Functions


```py

def greet_user():
    """Display a simple greeting."""
    print("Hello!")

greet_user()

```

Above is a simple function definition, notice the docstring starting """ """ , this is used by python when it generates documentation of your function .



### `__init__`  what is it ?

It is used to create critical components of a `Class` , for example in the Class
below `name` and `gender` are critical components of a `Human`

```py

class Human():
    def __init__(self, name, gender):
        self.name = name
        self.gender = gender

Will =  Human('Willie', 'Male')
print Will.name

```

`self` refers the object you created . So in the above example `Will` is the object you created. Hence `self` is reffering to `Will`.
`Will.name` and `Will.gender` are the object variables.
