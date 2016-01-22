---
title: What does if __name__ == '__main__'  do ?
tags:
  - python
date: 2015-02-10 19:45:46
---

在python经常能看到`if __name__ == '__main__: '`这一判断句，而且双下划线就意味着是内置的变量。那这句话到底有什么作用？去stackoverflow上搜了下看到一个不错的回答[[python - What does `if __name__ == &quot;__main__&quot;:` do? - Stack Overflow]](http://http://stackoverflow.com/questions/419163/what-does-if-name-main-do "python - What does `if __name__ == &quot;__main__&quot;:` do? - Stack Overflow")，这个判断句的作用也就了解了。
我大致的理解：
当python的解释器读取一个源文件的时候，解释器会在真正执行代码之前把文件中的代码执行一遍。在这个过程中，解释器会定义一些特殊的内部变量。例如，解释器直接运行该代码的时候会定义变量`__name__`，并把值`'__main__'`赋给它;相反，如果，这个代码正在被另一个python程序`import`的时候，那么解释器就会把`__name__`这个变量的值赋成代码文件本身的名字'**'(代码文件名为: **.py)。

写了几行测试代码，大概解释一下：
文件名: test__name__.py

``` python
import sys #just for explaining the process

def test():
    print "test"

print __name__
if __name__ == '__main__':
    print "run me"

if __name__ == 'test__name__':
    print "import me"
```

当在shell中执行python test.py的时候，解释器设置一些特殊的变量，例如`__name__ == '__main__'`.之后，他会导入`sys`模块，在之后会读取`def`函数块，创建一个函数object，并创建一个名为'test'的变量引用这个函数object. 在之后会读if语句，好了，关键到了，由于现在是直接在命令行python test.py, 这时`__name__ == 'test__name__'`的返回值为`True`，那自然会执行`print "run me"`. 另一种情况，就是我在命令行执行`import test__name__`, 在读取if语句之前的过程和之前相同，读到if语句的时候会进行判断，这个时候由于我不是直接执行代码，而是把文件当作模块导入，解释器附给变量'`__name__`'的值不再是`'__main__'`了而是文件的名字`'test__name__'`, 便会执行`print "import me"`.

执行结果:
``` python
In [4]: run test__name__.py
__main__
run me

In [5]: import test__name__
test__name__
import me
```

这一语句有什么作用? 其中一个作用就是当导入代码的时候我们很多时候并不想执行某些语句，有时候有些print语句会在命令行显示出来，这样这条判断句就可以导入代码的时候不执行某些语句。
回答里的原话:

> One of the reasons for doing this is that sometimes you write a module (a .py file) where it can be executed directly. Alternatively, it can also be imported and used in another module. By doing the main check, you can have that code only execute when you want to run the module as a program and not have it execute when someone just wants to import your module and call your functions themselves.
