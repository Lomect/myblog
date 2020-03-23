---
title: pytest
date: 2020-03-23 23:25:43
tags: 
    - Test
    - Python
mathjax: true
---

### Install
```shell script
pip install pytest
```
### 测试信息打印
```shell script
-r:
    -a 除了`pass`打印所有
    -f 失败的信息
    -E error
    -s skipped
    -x xfailed
    -X xpass
    -p pass
    -P passed with output
    -a all except pP 
```

- 测试失败后自动调用`pdb` 
```
pytest --pdb
pytest -x --pdb // 首次失败后调用pdb，然后结束测试进程
pytest --pdb --maxfail=3 // 第三次失败的时候调用pdb
pytest --trace // 测试启动后立即调用pdb
# 代码中可是以使用
import pdb
pdb.set_trace() // 设置断点
```

- 创建 JUnitXML 格式的文件
可以创建jenkins或其他集成测试软件能够读取的文件
```shell script
pytest --junitxml=path
```
- 可以在`pytest`的配置文件中设置junit_suite_name来配置测试结果中XML中的root名称
```shell script
[pytest]
junit_suite_name = my_suite
```
- junit xml规范中的时间是总时间，想要只显示测试时间，配置`junit_duration_report`:
```shell script
[pytest]
junit_duration_report = call
```

### 断言
- 标准`python`断言
```python
def f():
    return 3

def test_f():
    assert f() == 4, "three is not eq four"
```
- 异常断言
可以在上下文中使用`pytest.raises`来对异常产生断言
```python
import pytest

def test_zero_div():
    with pytest.raises(ZeroDivisionError) as zero:
        1/0
        assert "Error" in zero.value
```
可以使用`pytest.mark.xfail`中使用`raises`来检测测试用例是否因为抛出异常而失败
```python
import pytest

@pytest.mark.xfail(IndexError)
def test_f():
    f()
```
使用`pytest.raises`更适合测试自己的代码故意引发的异常， 使用`@pytest.mark.xfail`和检查函数更适合那些记录的未被修复的bug
或者依赖项的异常

- 为断言增加描述
可以使用`pytest_assertrepr_compare(config, op, left, right)` 为自己的断言增加描述

### fixture
`pytest.fixture`装饰的函数可以作为测试函数的入参
```python
import pytest

@pytest.fixture
def foo():
    return 5

def test_foo(foo):
    assert foo == 5
```

- fixfure: 依赖注入的最佳实践

`pytest`中的`fixture`允许测试函数轻松的接受和处理应用层对象的预处理，而不必关心`import/setup/teardown`这些细节，
这是依赖注入的一个极佳的示范，`fixture`是注入器，而测试函数是`fixture`的调用者

- conftest.py 共享fixture函数

在实现测试用例的过程中，当发现需要使用来自多个文件的`fixture`函数的时候，可以将这些函数放入`conftest.py`文件中
这样就可以不用主动导入`fixture`函数，`pytest`会自动检索。可以使用`conftest.py`为每个目录实现插件。

- scope： 在类/模块/函数/整个测试用例中共享`fixture`实例

scope： function， classs， module， package，session
scope越大，实例化越早。

可以用`getatter`来获取到测试对象的上下文
```python
import pytest

abs = "abs"
@pytest.fixture
def get_abc(request):
    param = getattr(request.module ,  "abs", "abc")
    assert param == "abs"
```
这里使用`request.module`属性来获取，模块中的`abs`属性。

- 工厂化的`fixture`

工厂化的`fixture`可以在单一的测试函数中被多次调用，可以用于返回数据。
如果需要还可以携带参数，下面是一个例子：
```python
import pytest

@pytest.fixture
def make_record():
    create_record = []
    def _make_record(name):
        record = models.Customer(name=name, recodes=[])
        create_record.append(record)
        return record
    yield  _make_record
    
    for record in create_record:
        record.destory()

def test_record(make_record):
    record1 = make_record("lomect")
    record2 = make_record("lome")
    record3 = make_record("lo")
```

- fixtures 参数化

`fixture`函数可以进行参数话的调用，这种情况下，相关的测试函数会被多次调用。该功能可以用来以多种方式配置的组件编写详尽
功能测试

```python
import pytest

@pytest.fixture(scope="module", params=["lome","lory"])
def get_data(request):
    yield request.param
    print("use data {}".format(request.param))
    print("deatory data")
```
- 在参数化的fixture中使用marks
```python
import pytest

@pytest.fixture(params=[0, 1, pytest.param(2, marks=pytest.mark.skip)])
def data_set(request):
    return request.param
```

- 模块化   
不仅测试函数可以使用`fixture`，`fixture`也可以使用`fixture`
```python


import pytest
import smtplib


@pytest.fixture(scope="module", params=["smtp.qq.com", "mail.python.org"])
def smtp_connection(request):
    smtp_connect = smtplib.SMTP(request.param, 587, timeout=3)
    yield smtp_connect
    print("{} {}", smtp_connect, request.param)
    smtp_connect.close()


class App(object):
    def __init__(self, smtp_connection):
        self.smtp = smtp_connection


@pytest.fixture(scope="module")
def app(smtp_connection):
    return App(smtp_connection)

def test_smtp_connection(app):
    assert app.smtp
```
session作用域的fixture不能引用一个module作用域的fixture。 即作用域大的`fixture`不能引用一个作用域小的`fixture`

- fixture的自动分组    
```python
import pytest


@pytest.fixture(scope="module", params=["mod1", "mod2"])
def modarg(request):
    param = request.param
    print("\nstart mod {}".format(param))
    yield param
    print("\nteardown mod {}".format(param))


@pytest.fixture(scope="function", params=[1, 2])
def funcarg(request):
    param = request.param
    print("\nstart func {}".format(param))
    yield param
    print("\nteardown func {}".format(param))


def test_0(funcarg):
    print("run test0 with funcarg {}".format(funcarg))


def test_1(modarg):
    print("run test1 with modarg {}".format(modarg))


def test_2(funcarg, modarg):
    print("run test2 with funcarg {} and modarg {}".format(funcarg, modarg))
```
日志如下：
```
main.py::test_0[1] 
start func 1
run test0 with funcarg 1
PASSED
teardown func 1

main.py::test_0[2] 
start func 2
run test0 with funcarg 2
PASSED
teardown func 2

main.py::test_1[mod1] 
start mod mod1
run test1 with modarg mod1
PASSED
main.py::test_2[mod1-1] 
start func 1
run test2 with funcarg 1 and modarg mod1
PASSED
teardown func 1

main.py::test_2[mod1-2] 
start func 2
run test2 with funcarg 2 and modarg mod1
PASSED
teardown func 2

main.py::test_1[mod2] 
teardown mod mod1

start mod mod2
run test1 with modarg mod2
PASSED
main.py::test_2[mod2-1] 
start func 1
run test2 with funcarg 1 and modarg mod2
PASSED
teardown func 1

main.py::test_2[mod2-2] 
start func 2
run test2 with funcarg 2 and modarg mod2
PASSED
teardown func 2

teardown mod mod2
```
从上面日志可以看出，大作用域的`fixtures`先被实例化，小作用域的`fixture`后被实例化，大作用域的`fixture`会将调用
它的作用域内的测试函数全部运行。

- 在class/module中使用fixture        
在`class`中使用`fixture`
```python
# conftest.py
import pytest
import tempfile
import os

@pytest.fixture()
def cleandir():
    newpath = tempfile.mkdtemp()
    os.chdir(newpath)

# test_setenv.py
import pytest
import os

@pytest.mark.usefixtures("cleandir")
class TestDirInit(object):
    def test_cwd_start_empty(self):
        assert os.listdir(os.getcwd()) == []
        with open("myfile", "w") as f:
            f.write("hello")

    def test_cwd_again_start_empty(self):
        assert os.listdir(os.getcwd()) == []
```
`cleandir`会在每个测试用例执行之前被调用。
也可以执行多个`fixtures`
```python
import pytest

@pytest.mark.usefixtures("cleandir", "otherfixture")
def test():
    pass
```
可以使用mark的通用特性来为测试module指定fixture：
```python
import pytest
pytestmark = pytest.mark.usefixtures("cleandir")
```
也可以把`fixture`写进配置文件中，对所有的测试用例起作用：
```editorconfig
# pytest.ini
[pytest]
usefixtures = cleandir
```
另外`usefixtures`对`fixture`不起作用。

- 自动使用`fixtures`     
假设有一个数据库`fixture`，包含`begin/rollback/commit`等操作，我们希望在测试用例的前后分别调用数据库的事务处理和rollback操作
```python
import pytest

class DB(object):
    def __init__(self):
        self.database = []
    def begin(self, name):
        self.database.append(name)
    def rollback(self):
        self.database.pop()

@pytest.fixture(scope="module")
def db():
    return DB()


class TestClass(object):
    @pytest.fixture(autouse=True)
    def transact(self, request, db):
        db.begin(request.function.__name__)
        yield
        db.rollback()

    def test_mod1(self, db):
        assert db.database == ["test_mod1"]

    def test_mod2(self, db):
        assert db.database == ["test_mod2"]
```
autouse的规则：       
    1. `autouse fixture`遵循`scope`的定义，如果scope为session，则，fixture无论在哪里都只会执行一次，
        若定义为class，则表示每个class执行一次。
    2. 若定义为module，则该模块所有的测试用例都会执行
    3. conftest中定义的autouse fixture 该目录下的测试用力都会执行

- 重写fixture
为了保证代码的可维护性可读性，可以重新定义fixture来重写，上层的fixture。
    - 在conftest层的重写
    - 在module层的重写
    - 直接使用参数话的重写
    ```python
    import pytest
    # conftest.py
    @pytest.fixture
    def username():
      return "username"
    
    # test_some.py
    @pytest.mark.parametrize("username", ["some-username"])
    def test_some(username):
      assert username == "some-username"
  
    
    def test_1_some(username):
        assert username == "username"
    ```
    - 可以用参数化的fixture来重写非参数化/参数化的fixture
    - 也可以用非参数化的fixture来重写非参数化/参数化的fixture
