---
title: python with的原理及在异常处理中的应用
tags: [Python]
copyright: true
date: 2019-09-11 11:37:54
permalink:
categories: Python
description: python with的原理及在异常处理中的应用
image: https://static001.geekbang.org/resource/image/cc/1f/cc6435160a6749ede9b70fc195b4241f.jpg
---
<p class="description"></p>
<img src="https://" alt="" style="width:100%" />

<!-- more -->

只要对象内实现 __enter__() 和 __exit__() 方法，就能兼容 with 语句，触发上下文管理。 

一、上下文管理的简单执行流程
with 工作原理 （１）紧跟with后面的语句被求值后，返回对象的“__enter__()”方法被调用，这个方法的返回值将被赋值给as后面的变量； （２）当with后面的代码块全部被执行完之后，将调用前面返回对象的“__exit__()”方法。

class Sample:
    def __enter__(self):
        print "in __enter__"
        return "Foo"
    def __exit__(self, exc_type, exc_val, exc_tb):
        '''
        若在执行流程中因为错误而退出，调用exit时，会自动捕获错误信息
        exc_type：　错误的类型(异常类型)
        exc_val：　错误类型对应的值 (异常值)
        exc_tb：　代码中错误发生的位置 (错误栈)
        '''
        print "in __exit__"
        
def get_sample():
    return Sample()
    
with get_sample() as sample:
    print "Sample: ", sample
'''
流程总结：
1- 执行get_sample()函数
2- 函数内实例化Sample对象，执行__enter__返回字符串赋予sample变量
3- 执行with内的代码块，输出sample变量值
4- 执行Sample对象内的__exit__方法
'''
二、错误执行流程
class Sample():
    def __enter__(self):
        print('in enter')
        return self
        
    def __exit__(self, exc_type, exc_val, exc_tb):
        print "type: ", exc_type
        print "val: ", exc_val
        print "tb: ", exc_tb
        
    def do_something(self):
        bar = 1 / 0
        return bar + 10

with Sample() as sample:
    sample.do_something()

'''
in enter
Traceback (most recent call last):
type:  <type 'exceptions.ZeroDivisionError'>
val:  integer division or modulo by zero
  File "/home/user/cltdevelop/Code/TF_Practice_2017_06_06/with_test.py", line 36, in <module>
tb:  <traceback object at 0x7f9e13fc6050>
    sample.do_something()
  File "/home/user/cltdevelop/Code/TF_Practice_2017_06_06/with_test.py", line 32, in do_something
    bar = 1 / 0
ZeroDivisionError: integer division or modulo by zero

Process finished with exit code 1

'''
三、异常处理 with 应用
异常处理逻辑太多，以至于扰乱了代码核心逻辑。具体表现就是，代码里充斥着大量的 try、except、raise 语句，让核心逻辑变得难以辨识。

def upload_avatar(request):
    """用户上传新头像"""
    try:
        avatar_file = request.FILES['avatar']
    except KeyError:
        raise error_codes.AVATAR_FILE_NOT_PROVIDED

    try:
       resized_avatar_file = resize_avatar(avatar_file)
    except FileTooLargeError as e:
        raise error_codes.AVATAR_FILE_TOO_LARGE
    except ResizeAvatarError as e:
        raise error_codes.AVATAR_FILE_INVALID
    
    try:
        request.user.avatar = resized_avatar_file
        request.user.save()
    except Exception:
        raise error_codes.INTERNAL_SERVER_ERROR
    return HttpResponse({})
这是一个处理用户上传头像的视图函数。这个函数内做了三件事情，并且针对每件事都做了异常捕获。如果做某件事时发生了异常，就返回对用户友好的错误到前端。

这样的处理流程纵然合理，但是显然代码里的异常处理逻辑有点“喧宾夺主”了。一眼看过去全是代码缩进，很难提炼出代码的核心逻辑。

早在 2.5 版本时，Python 语言就已经提供了对付这类场景的工具：“上下文管理器（context manager）”。上下文管理器是一种配合 with 语句使用的特殊 Python 对象，通过它，可以让异常处理工作变得更方便。

class raise_api_error:
    """
    captures specified exception and raise ApiErrorCode instead
    捕获指定的异常并改为引发ApiErrorCode
    :raises: AttributeError if code_name is not valid
    """
    def __init__(self, captures, code_name):
        self.captures = captures
        self.code = getattr(error_codes, code_name)

    def __enter__(self):
        # 刚方法将在进入上下文时调用
        return self
    
    def __exit__(self, exc_type, exc_val, exc_tb):
        # 该方法将在退出上下文时调用
        # exc_type, exc_val, exc_tb 分别表示该上下文内抛出的
        # 异常类型、异常值、错误栈
        if exc_type is None:
            return False
    
        if exc_type == self.captures:
            raise self.code from exc_val
        return False
在上面的代码里，我们定义了一个名为 raise_api_error 的上下文管理器，它在进入上下文时什么也不做。但是在退出上下文时，会判断当前上下文中是否抛出了类型为 self.captures 的异常，如果有，就用 APIErrorCode 异常类替代它。

使用该上下文管理器后，整个函数可以变得更清晰简洁：

def upload_avatar(request):
    """用户上传新头像"""
    with raise_api_error(KeyError, 'AVATAR_FILE_NOT_PROVIDED'):
        avatar_file = request.FILES['avatar']

    with raise_api_error(ResizeAvatarError, 'AVATAR_FILE_INVALID'),\
            raise_api_error(FileTooLargeError, 'AVATAR_FILE_TOO_LARGE'):
        resized_avatar_file = resize_avatar(avatar_file)
    
    with raise_api_error(Exception, 'INTERNAL_SERVER_ERROR'):
        request.user.avatar = resized_avatar_file
        request.user.save()
    return HttpResponse({})
四、raies 和 raise……from 的区别
>>> try:
>>> ...     print(1 / 0)
>>> ... except:
>>> ...     raise RuntimeError("Something bad happened")
>>> ...


Traceback (most recent call last):
  File "<stdin>", line 2, in <module>
ZeroDivisionError: division by zero

#  在处理上述异常期间，发生了另一个异常：
During handling of the above exception, another exception occurred:

Traceback (most recent call last):
  File "<stdin>", line 4, in <module>
RuntimeError: Something bad happened
>>> try:
>>> ...     print(1 / 0)
>>> ... except Exception as exc:
>>> ...     raise RuntimeError("Something bad happened") from exc
>>> ...

Traceback (most recent call last):
  File "<stdin>", line 2, in <module>
ZeroDivisionError: division by zero

# 上述异常是以下异常的直接原因：
The above exception was the direct cause of the following exception:

Traceback (most recent call last):
  File "<stdin>", line 4, in <module>
RuntimeError: Something bad happened
不同之处在于，from 会为异常对象设置 _cause_ 属性表明异常的是由谁直接引起的。

处理异常时发生了新的异常，在不使用 from 时更倾向于新异常与正在处理的异常没有关联。而 from 则是能指出新异常是因旧异常直接引起的。这样的异常之间的关联有助于后续对异常的分析和排查。from 语法会有个限制，就是第二个表达式必须是另一个异常类或实例。

如果在异常处理程序或 finally 块中引发异常，默认情况下，异常机制会隐式工作会将先前的异常附加为新异常的 _context _属性。

当然，也可以通过 with_traceback() 方法为异常设置上下文 _context_ 属性，这也能在 traceback 更好的显示异常信息。

<hr />
