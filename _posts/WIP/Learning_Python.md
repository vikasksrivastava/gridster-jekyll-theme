# Day 1 - Target 20 Pages


Zen of Python


```py
import this
```
```
Complex is better than complicated.
Flat is better than nested.
Sparse is better than dense.
Readability counts.
```

A comment can be made anywhere with `#`
```py
# This is a comment
print(Hello) # This is a comment too !
```

Basic Example of a Variable (in this case **message**)

```py
message = "Hello!"
print(message)
```

Basic String Manipulation

```py
message = "Hello!"
print(message.upper())
```

Combining or Concatenating Strings

```py
message1 = "Hello"
message2 = "World!"
print(message1 + message2)
```

Stripping White Space (`rstrip,lstrip,strip`)

```py
message1 = "Hello   "
print(message1.rstrip())
```

Escape Sequence

```py
message1 = "Hello \"World   "
print(message1)
```

**Avoiding type errors with str() functions**

```py
age = 23
message = "Happy" + age + "birthday"
```

The above code will lead to type error as python is unable to know if  `age` int or str. So we need to define like this:

```py
age = 23
message = "Happy" + str(age) + "birthday"
```

**Python List**

List position index starts at `0` not `1`

```py
vegetables = [`tomato`, `brocolli`, `carrot`]
print(vegetables)
```

The above will print the representation of list with the brackets

```
[`tomato`, `brocolli`, `carrot`]
```

Accessing one of the items of the list

```py
vegetables = [`tomato`, `brocolli`, `carrot`]
print(vegetables[2])
```

Prints `carrot`

**Changing, adding, and removing elements** PAGE 73
