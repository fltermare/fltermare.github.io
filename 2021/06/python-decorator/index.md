# Python - decorator


## Introduction
紀錄一下， decorator 怎麼用  
個人比較常使用到的情境是紀錄各個步驟(function)使用的時間 (e.g. pre-processing / training / inference ...)  

透過 decorator 達到 1. 減少重複 code 2. 增加可讀性 的目的

* full sample code:
  * https://gist.github.com/fltermare/32af9e5abf6bb0cd6e230ee755ee2f60

### functools.wraps
使用 function 作為 decorator 時，記得要加上 `functools.wraps` 來重新包裝 func  
避免 `__name__` & `__doc__` 跟預期的不同  

以下是 https://docs.python.org/3/library/functools.html#functools.wraps 的範例
```py
# with wraps
from functools import wraps
def my_decorator(f):
    @wraps(f)
    def wrapper(*args, **kwds):
        print('Calling decorated function')
        return f(*args, **kwds)
    return wrapper

@my_decorator
def example():
    """Docstring"""
    print('Called example function')

>>> example()
Calling decorated function
Called example function
>>> example.__name__
'example'
>>> example.__doc__
'Docstring'
```
沒有的話
```py
# without wraps
def my_decorator(f):
    def wrapper(*args, **kwds):
        print('Calling decorated function')
        return f(*args, **kwds)
    return wrapper

@my_decorator
def example():
    """Docstring"""
    print('Called example function')

>>> example()
Calling decorated function
Called example function
>>> example.__name__
'wrapper'
>>> example.__doc__
>>>
```

### Order - multiple decorators
```py
def decorator_c(func):
    @wraps(func)
    def wrapper(*args, **kargs):
        print('In decorator_c')
        return func(*args, **kargs)
    return wrapper


def decorator_b(func):
    @wraps(func)
    def wrapper(*args, **kargs):
        print('In decorator_b')
        return func(*args, **kargs)
    return wrapper


def decorator_a(func):
    @wraps(func)
    def wrapper(*args, **kargs):
        print('In decorator_a')
        return func(*args, **kargs)
    return wrapper


@decorator_a
@decorator_b
@decorator_c
def order_func():
    print('In order_func')

>>> order_func()
In decorator_a
In decorator_b
In decorator_c
In order_func
```
從上而下的開始執行，`decorator_a -> decorator_b -> decorator_c`

## Function without Args
```py
def decorator_func_without_args(func):
    @wraps(func)
    def wrapper(*args, **kargs):
        start_time = time.time()
        print("{} Calling [{}]".format(start_time, func.__name__))

        res = func(*args, **kargs)

        return res
    return wrapper


@decorator_func_without_args
def dummy_func_a(msg):
    """doc aaaaaaa"""

    time.sleep(1)
    return "msg: {}\n".format(msg)


print(dummy_func_a("Called dummy_func_a"))
```

```sh
1622879790.3779058 Calling [dummy_func_a]
msg: Called dummy_func_a
```

## Function with Args
```py
def decorator_func_with_args(decorator_args):
    def outer_wraper(func):
        @wraps(func)
        def wrapper(*args, **kargs):
            start_time = time.time()
            print("{} Calling [{}]".format(start_time, func.__name__))
            print("[!] get: '{}'".format(decorator_args))

            res = func(*args, **kargs)

            return res
        return wrapper
    return outer_wraper


@decorator_func_with_args("args for decorator_func_with_args")
def dummy_func_b(msg):
    """doc bbbbbbb"""

    time.sleep(1)
    return "msg: {}\n".format(msg)


print(dummy_func_a("Called dummy_func_b"))
```

```sh
1622879791.3789802 Calling [dummy_func_b]
[!] get: 'args for decorator_func_with_args'
msg: Called dummy_func_b
```

## Class without Args
```py
class DecoratorWithoutArgs:
    def __init__(self, func):
        self.func = func

    def __call__(self, *args, **kargs):
        start_time = time.time()
        print("{} Calling [{}]".format(start_time, self.func.__name__))

        res = self.func(*args, **kargs)

        return res


@DecoratorWithoutArgs
def dummy_func_c(msg):
    """doc ccccccc"""

    time.sleep(1)
    return "msg: {}\n".format(msg)


print(dummy_func_c("Called dummy_func_c"))
```

```sh
1622879792.3800542 Calling [dummy_func_c]
msg: Called dummy_func_c

```

## Class with Args
```py
class DecoratorWithArgs:
    def __init__(self, decorator_args):
        self.decorator_args = decorator_args

    def __call__(self, func):
        def wrapper(*args, **kargs):
            start_time = time.time()
            print("{} Calling [{}]".format(start_time, func.__name__))
            print("[!] get: '{}'".format(self.decorator_args))

            res = func(*args, **kargs)

            return res
        return wrapper


@DecoratorWithArgs("args for decorator_func_with_args")
def dummy_func_d(msg):
    """doc ddddddd"""

    time.sleep(1)
    return "msg: {}\n".format(msg)


print(dummy_func_d("Called dummy_func_d"))
```

```sh
1622879793.3805652 Calling [dummy_func_d]
[!] get: 'args for decorator_func_with_args'
msg: Called dummy_func_d
```

## Inside Class [?]
```py
class DummyClass:
    def __init__(self):
        self.para = "para in __init__"

    def decorator_func_inclass(func):
        @wraps(func)
        def wrapper(*args, **kargs):
            start_time = time.time()
            print("{} [{}] start".format(start_time, func.__name__))

            #####
            self = args[0]
            print("[!] get: '{}'".format(self.para))
            #####

            res = func(*args, **kargs)
            return res
        return wrapper

    @decorator_func_inclass
    def dummy_func_e(self, msg):
        """doc eeeeeee"""

        time.sleep(1)
        return "msg: {}\n".format(msg)


obj = DummyClass()
print(obj.dummy_func_e("Called dummy_func_e"))
```

```sh
1622879794.3816772 [dummy_func_e] start
[!] get: 'para in __init__'
msg: Called dummy_func_e
```

這個例子比較特別，因為某些原因我想在 decorator 中使用 object 的 attribute  
因此需要能使用到 `self`，後來找到這個作法 (ref 的第 4 個 URL)  
不確定是否正確，有錯請指正

## Conclusion
個人的感想是
- 如果不用 args 就選 [function without args](#function-without-args)
- 如果需要 args 就選 [class with args](#class-with-args)

## Ref
1. [Python Decorator 入門教學](https://blog.techbridge.cc/2018/06/15/python-decorator-%E5%85%A5%E9%96%80%E6%95%99%E5%AD%B8/)
2. [快速理解並使用Python Decorator(裝飾器)](https://blog.typeart.cc/%E5%BF%AB%E9%80%9F%E7%90%86%E8%A7%A3%E4%B8%A6%E4%BD%BF%E7%94%A8Python%20Decorator/)
3. [[Python教學] 裝飾詞原理到應用](https://www.maxlist.xyz/2019/12/07/python-decorator/)
4. https://stackoverflow.com/questions/1263451/python-decorators-in-classes

