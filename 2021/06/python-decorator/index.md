# Python - decorator


## Introduction
紀錄一下， decorator 怎麼用  
個人比較常使用到的情境是紀錄各個步驟(function)使用的時間 (e.g. pre-processing / training / inference ...)

個人的感想是
- 如果不用 args 就選 [function without args](#function-without-args)
- 如果需要 args 就選 [class with args](#class-with-args)

* full sample code:
  * https://gist.github.com/fltermare/32af9e5abf6bb0cd6e230ee755ee2f60

## function without args
```python
def decorator_func_without_args(func):
    @wraps(func)
    def wrapper(*args, **kargs):
        start_time = time.time()
        print("{} [{}] start".format(start_time, func.__name__))

        res = func(*args, **kargs)

        end_time = time.time()
        print("{} [{}] end, cost: {}".format(end_time, func.__name__, end_time - start_time))
        print("[{}] doc: '{}'".format(func.__name__, func.__doc__))

        return res
    return wrapper


@decorator_func_without_args
def dummy_func_a(msg):
    """doc aaaaaaa"""

    time.sleep(1)

    return "msg: {}\n".format(msg)


print(dummy_func_a("args -  dummy_func_a"))
```

```sh
1622875328.5369604 [dummy_func_a] start
1622875329.5382118 [dummy_func_a] end, cost: 1.001251459121704
[dummy_func_a] doc: 'doc aaaaaaa'
msg: args -  dummy_func_a
```


## function with args
```python
def decorator_func_with_args(decorator_args):
    def outer_wraper(func):
        @wraps(func)
        def wrapper(*args, **kargs):
            start_time = time.time()
            print("{} [{}] start".format(start_time, func.__name__))

            #####
            print("[!] {}".format(decorator_args))
            #####

            res = func(*args, **kargs)

            end_time = time.time()
            print("{} [{}] end, cost: {}".format(end_time, func.__name__, end_time - start_time))
            print("[{}] doc: '{}'".format(func.__name__, func.__doc__))

            return res
        return wrapper
    return outer_wraper


@decorator_func_with_args("args for decorator_func_with_args")
def dummy_func_b(msg):
    """doc bbbbbbb"""

    time.sleep(1)

    return "msg: {}\n".format(msg)


print(dummy_func_b("args -  dummy_func_b"))
```

```sh
1622875329.538264 [dummy_func_b] start
[!] args for decorator_func_with_args
1622875330.539237 [dummy_func_b] end, cost: 1.0009729862213135
[dummy_func_b] doc: 'doc bbbbbbb'
msg: args -  dummy_func_b
```

## class without args
```python
class DecoratorWithoutArgs:
    def __init__(self, func):
        self.func = func

    def __call__(self, *args, **kargs):
        start_time = time.time()
        print("{} [{}] start".format(start_time, self.func.__name__))

        res = self.func(*args, **kargs)

        end_time = time.time()
        print("{} [{}] end, cost: {}".format(end_time, self.func.__name__, end_time - start_time))
        print("[{}] doc: '{}'".format(self.func.__name__, self.func.__doc__))

        return res


@DecoratorWithoutArgs
def dummy_func_c(msg):
    """doc ccccccc"""

    time.sleep(1)

    return "msg: {}\n".format(msg)


print(dummy_func_c("args -  dummy_func_c"))
```

```sh
1622875330.5393002 [dummy_func_c] start
1622875331.540669 [dummy_func_c] end, cost: 1.001368761062622
[dummy_func_c] doc: 'doc ccccccc'
msg: args -  dummy_func_c
```

## class with args
```python
class DecoratorWithArgs:
    def __init__(self, decorator_args):
        self.decorator_args = decorator_args

    def __call__(self, func):
        def wrapper(*args, **kargs):
            start_time = time.time()
            print("{} [{}] start".format(start_time, func.__name__))

            #####
            print("[!] {}".format(self.decorator_args))
            #####

            res = func(*args, **kargs)

            end_time = time.time()
            print("{} [{}] end, cost: {}".format(end_time, func.__name__, end_time - start_time))
            print("[{}] doc: '{}'".format(func.__name__, func.__doc__))

            return res
        return wrapper


@DecoratorWithArgs("args for decorator_func_with_args")
def dummy_func_d(msg):
    """doc ddddddd"""

    time.sleep(1)

    return "msg: {}\n".format(msg)


print(dummy_func_d("args -  dummy_func_d"))
```

```sh
1622875331.5407255 [dummy_func_d] start
[!] args for decorator_func_with_args
1622875332.5420897 [dummy_func_d] end, cost: 1.0013642311096191
[dummy_func_d] doc: 'doc ddddddd'
msg: args -  dummy_func_d
```
## inside class
```python
class DummyClass:
    def __init__(self):
        self.para = "para in init"

    def decorator_func_inclass(func):
        @wraps(func)
        def wrapper(*args, **kargs):
            start_time = time.time()
            print("{} [{}] start".format(start_time, func.__name__))

            #####
            self = args[0]
            print("[!] {}".format(self.para))
            #####

            res = func(*args, **kargs)

            end_time = time.time()
            print("{} [{}] end, cost: {}".format(end_time, func.__name__, end_time - start_time))
            print("[{}] doc: '{}'".format(func.__name__, func.__doc__))

            return res
        return wrapper

    @decorator_func_inclass
    def dummy_func_e(self, msg):
        """doc eeeeeee"""

        time.sleep(1)

        return "msg: {}\n".format(msg)


obj = DummyClass()
print(obj.dummy_func_e("args -  dummy_func_e"))
```

```sh
1622875332.5421455 [dummy_func_e] start
[!] para in init
1622875333.5435815 [dummy_func_e] end, cost: 1.0014359951019287
[dummy_func_e] doc: 'doc eeeeeee'
msg: args -  dummy_func_e
```

這個例子比較特別，因為某些原因我想在 decorator function 中使用 object 的 attribute  
因此需要能使用到 `self`，後來找到這個作法 (ref 的第二個 URL)  
不確定是否正確，有錯請指正

## Ref
* [Python Decorator 入門教學](https://blog.techbridge.cc/2018/06/15/python-decorator-%E5%85%A5%E9%96%80%E6%95%99%E5%AD%B8/)
* https://stackoverflow.com/questions/1263451/python-decorators-in-classes

