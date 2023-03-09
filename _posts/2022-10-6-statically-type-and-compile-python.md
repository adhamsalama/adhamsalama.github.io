---
title: Statically type & compile Python code
date: 2022-10-06 12:12:12 +/-0800
categories: []
tags: [python] # TAG names should always be lowercase
author: adham
---

Python is a dynamically typed, interpreted programming language, unlike statically typed, compiled languages like C++. _But it can be!_

You can write Python code in a statically typed way using type hints, check for type errors using [mypy](http://mypy-lang.org), and you can compile any Python using [Nuitka](https://nuitka.net).
So, if you mix them both, you get statically typed, compiled Python code!

Let's see how we can achieve this.

1. Install mypy.

- `pip install mypy`

2. Install Nuitka.

- `pip install nuitka` (You may need to install python3-devel first)

That's it. Now we can start writing our code.

## Example

Let's define a Person class, that has a name that is a string, an age that is an int, a job that is a string, and a salary that is a float.

The code would look like this. Create a file called main.py (or any name you want).

```python
class Person:
    name: str
    age: int
    job: str
    salary: float

    def __init__(self, name: str, age: int, job: str, salary: float) -> None:
        self.name = name
        self.age = age
        self.job = job
        self.salary = salary

    def increase_salary(self, amount: float) -> None:
        self.salary += amount

    def __str__(self) -> str:
        return f"Hello, my name is {self.name}, I am {self.age} years old, and I am a {self.job}."


adham: Person = Person(name="Adham", age=23, job="Engineer", salary=10000)

amount: float = 5000

adham.increase_salary(amount)

print(adham)
```

Run "python main.py" in the terminal.

![Run using Interpreter](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/hukd8lhbsfu6ntxmou13.png)

We have used type hints to specify the types, but Python doesn't enforce these types, you can give a str a value of an int and Python wouldn't throw any errors.
For example, change the value of the variable amount to a string.
![Invalid type](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/tuk79wxwp7y5483r88fj.png)
Here my code editor gave me a hint that the type is incompatible, but Python will run the code, it will only throw an error at runtime when it executes the icrease_salary method.

![Runtime error](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/wwtfjf4xenio6fjmlgvy.png)
So how to catch these type errors without executing the code?
This is where mypy comes into play.

Run "mypy main.py" in the terminal.

![mypy invalid type check](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/73vgwb7wevrjkm978ul6.png)

You can see that mypy caught the error.
If we change the value of amount back to 5000 and run mypy again, here is the output.

![mypy correct type check](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/y4l9um79okba66j1r48c.png)

Now that we are sure that there aren't any type errors in our code, we can finally compile out code using Nuitka.

Run "nuitka3 main.py" in the terminal.

![Compile Python Code](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/63x09k9fabn5ze772grh.png)

Run "./main.bin" in the terminal to run the compiled code.

![Run compiled code](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/1bh6ctlyipw9bx11oyk6.png)

A slightly easier way to type check and compile our code in the same command is running "mypy main.py && nuitka3 main.py".

This will type check our code first, and if it's correct, it will get compiled.

That's it! Now you have Python++!
