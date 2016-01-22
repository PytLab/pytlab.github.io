---
title: 利用PyUnit framework写单元测试
tags:
  - python
  - unittest
date: 2015-02-10 16:35:06
---

以前写代码没有意识到单元测试的重要性。直到project的规模慢慢变大的时候，或者当把程序给别人使用抛出exception，自己着急要解决问题的时候才意识到提前写好unit test是很重要的(我理解的对么??)。于是慢慢的我也开始写unit test了。今天就看了下python的document上面介绍PyUnit的部分，简单的学习了一下利用PyUnit写单元测试脚本。
关于unit test,

> 在计算机编程中，单元测试（又称为模块测试, Unit Testing）是针对程序模块（软件设计的最小单位）来进行正确性检验的测试工作。程序单元是应用的最小可测试部件。在过程化编程中，一个单元就是单个程序、函数、过程等；对于面向对象编程，最小单元就是方法，包括基类（超类）、抽象类、或者派生类（子类）中的方法。> 
> 通常来说，程序员每修改一次程序就会进行最少一次单元测试，在编写程序的过程中前后很可能要进行多次单元测试，以证实程序达到软件规格书要求的工作目标，没有程序错误；虽然单元测试不是什么必须的，但也不坏，这牵涉到项目管理的政策决定。

python内部自带了一个单元测试的模块，PyUnit也就是我们说的：unittest. Official Doc上关于PyUnit的介绍: 
> The Python unit testing framework, sometimes referred to as “PyUnit,” is a Python language version of JUnit, by Kent Beck and Erich Gamma. JUnit is, in turn, a Java version of Kent’s Smalltalk testing framework. Each is the de facto standard unit testing framework for its respective language.> 
> 
> unittest supports test automation, sharing of setup and shutdown code for tests, aggregation of tests into collections, and independence of the tests from the reporting framework. The unittest module provides classes that make it easy to support these qualities for a set of tests.> 
> 
> To achieve this, unittest supports some important concepts:> 
> 
> **test fixture**> 
> A test fixture represents the preparation needed to perform one or more tests, and any associate cleanup actions. This may involve, for example, creating temporary or proxy databases, directories, or starting a server process.> 
> **test case**> 
> A test case is the smallest unit of testing. It checks for a specific response to a particular set of inputs. unittest provides a base class, TestCase, which may be used to create new test cases.> 
> **test suite**> 
> A test suite is a collection of test cases, test suites, or both. It is used to aggregate tests that should be executed together.> 
> **test runner**> 
> A test runner is a component which orchestrates the execution of tests and provides the outcome to the user. The runner may use a graphical interface, a textual interface, or return a special value to indicate the results of executing the tests.
由于是初次使用PyUnit框架写单元测试脚本，我还是用了简单的测试用例(test case),用了之后才发现，诶！不错～
unittest的简单使用，大概分以下几个方面：

下面是unittest模块的常用方法：

今天就用unittest为model的load()方法解析化学方程式写了单元测试，感觉爽爽的(๑>◡<๑) ,贴上代码:
``` python
import unittest
import sys

class TestKineticModel(unittest.TestCase):

    def setUp(self):
        #create an instance of KineticModel
        sys.path.append('E:\BaiduYun\MyDocuments\ECUST+\Python\script')
        from kinetic import model
        self.km = model.KineticModel(setup_file='methanation.mkm')

    def test_set_logger(self):
        "make sure set_logger method takes effects"
        self.assertTrue(hasattr(self.km, 'logger'))

        #should raise an exception for an AttributeError
        self.assertRaises(AttributeError)

    def test_load(self):
        """test the load() method has loaded variables
        in setup file into model as attrs of it"""

        #make sure all vars in setup file are parsed in
        globs, locs = {}, {}
        execfile(self.km.setup_file, globs, locs)
        for var in locs:
            self.assertTrue(hasattr(self.km, var))
            self.assertRaises(AttributeError)

    def test_parse_site_expression(self):
        "make sure site can be parsed successfully"
        site_expression = '3*_s'
        site_dict = self.km.parse_site_expression(site_expression)
        target_dict = {'s': {'number': 3, 'type': 's'}}
        self.assertDictEqual(target_dict, site_dict)

    def test_parse_species_expression(self):
        "make sure species expression e.g. '2CH3_s' can be parsed successfully"

        #test adsorbate
        adsorbate_expression = '2CH3_s'
        adsorbate_dict = \
                self.km.parse_species_expression(adsorbate_expression)
        target_adsorbate_dict = {'CH3_s': {'number': 2, 
                                 'site': 's', 
                                 'elements': {'C': 1, 'H': 3}}}
        self.assertDictEqual(adsorbate_dict, target_adsorbate_dict)

        #test transition state
        ts_expression = '3CH2-H_s'
        ts_dict = self.km.parse_species_expression(ts_expression)
        target_ts_dict = {'CH2-H_s': {'number': 3, 
                                      'site': 's', 
                                      'elements': {'C': 1, 'H': 3}}}
        self.assertDictEqual(ts_dict, target_ts_dict)

#if __name__ == '__main__':
#    unittest.main()
suite = unittest.TestLoader().loadTestsFromTestCase(TestKineticModel)
unittest.TextTestRunner(verbosity=2).run(suite)
```
运行脚本后可以显示每个case的测试结果，以及测试消耗时间。
``` python
In [2]: run model_test.py
test_load (__main__.TestKineticModel)
test the load() method has loaded variables ... ok
test_parse_site_expression (__main__.TestKineticModel)
make sure site can be parsed successfully ... ok
test_parse_species_expression (__main__.TestKineticModel)
make sure species expression e.g. '2CH3_s' can be parsed successfully ... ok
test_set_logger (__main__.TestKineticModel)
make sure set_logger method takes effects ... ok

----------------------------------------------------------------------
Ran 4 tests in 0.013s

OK
```
