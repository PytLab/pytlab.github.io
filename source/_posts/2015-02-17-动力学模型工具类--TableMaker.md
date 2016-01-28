---
title: 动力学模型工具类--TableMaker
tags:
  - kinetic model
  - kinetics
  - python
  - Kynetix
categories:
  - 学术
  - 代码作品
date: 2015-02-17 22:01:42
---

table_maker类本来是没有考虑到最初的模型框架中的，写到parser部分的时候在想可以根据setup file提供的基元反应化学方程式自动生成input file表格，这样方便以后使用本模型的用户在输入能量信息的时候能够根据模型的需求输入相应的能量数据，这样在输入数据的时候就避免了过多的输入或者漏掉数据的情况。
input file 主要形式是表格，其中header包含: 

`surface_name, site_name, species_name, DFT_energy, formation_energy, frequencies, infomation`

*   `surface_name`: 是研究的表面的名字，例如'Pd'; 如果是气体，则为None
*   `site_name`: 是指具体晶面，例如'111'
*   `species_name`: 是指吸附在表面的物种名称，如果是表面则为slab
*   `DFT_energy`: 是DFT计算出的初始能量，后面会对此能量进行处理，计算出generalized formation energy作为动力学计算的输入数据
*   `frequencie`: 振动频率，用于热力学校正，以list的形式写在表格中，数目根据分子的构型3N-5 或者 3N-6个
*   `information`: 数据的参考信息

<!-- more -->

自动生成表格的前提是model必须要有具有正确格式和信息的setup file，因为此table_maker生成表格的原理是，根据setup file生成临时的动力学模型对象，利用工具中的table_maker生成此对象对应的表格。
在CatMap中input file使用Tab分割的，不容易查看能量数据。在此模型中，我用了csv格式文件作为输入文件，引文都好分隔符文件是可以用excel打开的，虽然用excel修改数据会造成数据精度丢失，但是有表格的话数据会有很好的可视性。修改数据的话还是建议用NotePad或者其他编辑器打开修改。
生成的csv表格大致如下：<p>
![](csv.gif)<p>
![](csv_excel.gif)

table_maker实现过程:
1\. 还是要有一个`TableBase`类，用于生成不同的table_maker的子类。目前，这个基类主要包含一些获取于model对象中的一些变量。这些变量都是用parser解析setupfile获取的，因此若要用table_maker，必须要有parser并且对setupfile进行了正确的解析。因此在model的`load()`中一开始我是把table_maker作为和parser等工具一样通过执行setupfile后遍历局部变量的，结果在实例化parser之前先循环到了table_maker，就抛出异常了。。。因此后面我又单独写了个`set_table_maker()`方法，
``` python
    def set_table_maker(self, maker_name):
        """
        Import table_maker and set the instance of it as attr of model 
        """
        #The 'BLACK MAGIC' is hacked from CatMap (catmap/model.py)
        #https://docs.python.org/2/library/functions.html#__import__
        basepath = os.path.dirname(
                    inspect.getfile(inspect.currentframe()))
        if basepath not in sys.path:
            sys.path.append(basepath)
        #from loggers import logger
        _module = __import__('table_makers', globals(), locals())
        maker_instance = getattr(_module, maker_name)(owner=self)
        setattr(self, 'table_maker', maker_instance)
```
在遍历局部变量的时候，进行了对table_maker的判断：
``` python
#ignore table maker which will be loaded after load()
if key == 'table_maker': 
    setattr(self, 'table_maker_name', locs[key])
    continue
```
方法是获取table_maker的名称(`string`)，然后在`load()`之后通过`set_table_maker()`方法对table_maker进行实例化。
2\. csv_maker
这里主要包含两部分，一部分是生成初始的表格，另一部分就是更新表格。
生成初始表格，就是根据模型的属性生成的表格框架，也就是有物种信息但是没有能量数据以及振动频率数据，这些数据需要手动输入，输入后利用csv_maker的`create_new_table()`方法对表格进行更新，也就是计算能量生成相应的generalized formation energy数据。其中更新的mode有两种，一种是'add'，用于第一次生成generalized formation energy的情况; 另一种是'update', 用于已经生成过formation energy，但是修改过raw_energy的情况。更新表格的代码如下：
``` python
    def create_new_table(self, mode):
        """
        Read initial input file, calculate generalized formation energy.
        Create a new input file containing 
        a column of generalized formation energy.
        """
        #get old table content
        f = open('./energy.csv', 'rU')
        lines_list = f.readlines()
        f.close()

        content_str = ''

        #modify header
        header_str = lines_list[0].strip(string.whitespace)
        if mode == 'add':
            header_list = header_str.split(',')
            header_list.insert(4, 'formation_energy')
            new_header_str = ','.join(header_list) + '\n'
            content_str += new_header_str
        if mode == 'update':
            content_str += header_str + '\n'

        #modify species part
        for row_str in lines_list[1:]:
            new_row_str = self.get_new_row(row_str, mode=mode) + '\n'
            content_str += new_row_str

        #create new input file
        f = open('./energy.csv', 'w')
        f.write(content_str)
        f.close()
```
