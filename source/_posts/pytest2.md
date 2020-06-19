---
title: pytest2
date: 2020-06-19 14:05:49
tags:
    - pytest
    - test
    - python
---
## pytest nopython 测试实现逻辑
### pytest 用例收集入口
```
def pytest_collect_file(path, parent):
    pass
    return YamlTestFile.from_parent(fspath=path, parent=parent)
```
这个钩子函数，是用例收集的入口，用来收集过滤测试用例文件

### pytest 用例收集
```
class YamlTestFile(File):
    pass

    def collect(self):
        pass
        yield TestCase.from_parent(self, name=f"{name}.{_id}", spec=suite)
```

File类，是真正的用例收集类， 需要实现collect方法， 并且返回TestCase实例。

### pytest用例执行
```
class TestCase(Item):

    def __init__(self, name, parent, spec):
        super().__init__(name, parent)
        self.spec = spec

    def runtest(self) -> None:
        pass
```
Item类，是真正执行用例的类，pytest_runtest_call钩子函数会调用这个类的，runtest方法，运行用例。

