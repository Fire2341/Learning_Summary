在高中阶段，学生考试成绩的分析是一个重要的而频繁的应用场景，快速、有效、精准的生成学生成绩的分析报告，是学情监控和开展个性化教学的前提。这个问题是基础性问题，实现的方法非常多，主要是基于Excel。在这里用python的pandas做一遍，体会一下办公自动化的乐趣。这篇文章以高中学生的成绩分析为背景，使用pandas（是一个基于numpy的python的数据分析包）对学生成绩进行分析。本文分为如下部分：

> 1. 全校成绩表的生成（虚构）；
> 2. 年级成绩分析；
> 3. 班级成绩分析；
> 4. 学科成绩分析；
> 5. 总结与思考。
> 
> 学生成绩分析是本文的场景，写这篇文章的目的是总结我参加华为云大数据挑战赛时对于pandas的学习体会，供大家参考，本文的源码地址：https://github.com/Fire2341/Learning_Summary。

在开始之前，导入numpy和pandas，按照习惯写成如下形式。如果没有这个模块，还是老规矩，使用pip install numpy和pip install pandas安装一下。


```python
import numpy as np
import pandas as pd
```

#### 一、全校成绩表的生成（虚构）
在开始之前，先生成我们的分析对象，学生成绩表，假设本次考试为理科班的摸底考试。学生成绩表包括：

1. **基本信息**，包括：学生姓名、学生年级、学生班级；
2. **学生成绩**，假设学生的成绩服从正态分布，生成的成绩包括如下科目：语文（150分）、数学（150分）、英语（150）、物理（100分）、化学（100分）、生物（100分），并计算总分。

生成的表格流程如下：先确定每个年级的班级数目，并随机生成各班人数（55-68人之间），由此计算得到全校人数。根据全校人数随机生成学生姓名，并在确定各科平均值和标准差后，根据正态分布规律随机生成各个学生的各科成绩，并计算每位学生的总分，以此获得一份总的成绩汇总表，主要代码如下。由于这部分代码较为冗长且不是主要部分，感兴趣的朋友可以点击源码查看。

```python
class_name, student_num = generate_class() # 生成班级信息
all_num = students_sum(student_num) # 生成全校学生总数
student_name_group = generate_student_name(all_num) # 生成全校学生名字
student_info = init_table(class_name, student_num, student_name_group) # 将年级、班级、学生信息初始化到表格中
student_list = get_list(all_num, student_info) #生成成绩汇总表
student_list.to_excel('学生成绩表2.xlsx') # 保存成绩表
```

取数据表的前5个来看，还真像那么回事（所有名字和成绩数据均为python随机生成，如有雷同，纯属巧合）。在生成的成绩表中，一共有3个年级，其中高一26个班，高二27个班，高三个23班，各班学生人数介于55-68人之间，全校一共4619名学生。

![](https://upload-images.jianshu.io/upload_images/12875160-1289679f4c2083b6.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

为了展现pandas的相较于excel的优越性，在下面的分析中，各部分使用的代码尽量不超过5行。

####  二、年级分析

##### 2.1 各年级的最低分、最高分、平均分和中位数
为了直观的反映各年级的整体教学情况，在这里计算各年级的各科最高分、最低分、平均分和中位数。在这一部分用到的函数主要是.groupby和.agg。groupby可以按年级分组，.agg能够对各年级各分组应用各个函数（求最大值、最小值、平均值、中位数）进行计算。


```python
subject_name = ['语文','数学','英语','物理','化学','生物','总分']
grade_analysis = student_list.groupby('年级')[subject_name].agg(['max','min','mean','median']).reset_index()
grade_analysis.head()
```

从结果来看，这份随机生成的成绩表，各个科目的都有人考满分，不符合实际情况，但是符合我的正态分布规律了……

![](https://upload-images.jianshu.io/upload_images/12875160-4091ca76edb2c900.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

##### 2.2 获取各年级的成绩前5的学生

不管哪个层级的学校，拔尖学生都在学校人才培养工作占有重要地位，而学习成绩可以在一个侧面反映拔尖学生的范围。在这里筛选各年级成绩排名前5的学生。这一部分用到的函数主要是.groupby和.sort_values。使用.groupby的.rank()获取年级排名，使用.sort_values按照年级和总分进行分组，使用.groupby的.get_group('高三')获得高三学生的年级排名，在这里展示高三前五名的情况。

```python
student_list['年级排名'] = student_list['总分'].groupby(student_list['年级']).rank(ascending=False).astype(int)
student_list.sort_values(['年级','总分'], ascending=False, inplace=True)
student_list.groupby('年级').get_group('高三').head()
```

第1名真是天选之子，随机生成的成绩都能有3科拿满分。

![](https://upload-images.jianshu.io/upload_images/12875160-ec43a9eff197dbfe.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

获取年级倒数前5的代码一样，把.sort_values()的ascending参数（是否升序）改为True就可以。

```python
student_list.sort_values(['年级','总分'], ascending=True, inplace=True)
student_list.groupby('年级').get_group('高三').head()
```

![](https://upload-images.jianshu.io/upload_images/12875160-35043ad75ed6684f.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

##### 2.3 数据保存
数据保存是一个重要的环节，毕竟学校的成绩分析报告是要打印出来发给各个年级主任、班主任的。使用.to_excel()导出到excel中，在这里导出整体情况，以及各个年级的前五和倒数前五的情况。

```python
grade_excel = pd.ExcelWriter(r'年级分析.xlsx')
grade_analysis.to_excel(grade_excel, sheet_name='整体情况')

grade_name = ['高一','高二','高三']
student_list.sort_values(['年级','总分'], ascending=False, inplace=True)
for name in grade_name:
    student_list.groupby('年级').get_group(name).head().to_excel(grade_excel, sheet_name=name+'-前五')

student_list.sort_values(['年级','总分'], ascending=True, inplace=True)
for name in grade_name:
    student_list.sort_values(['年级','总分'], ascending=False, inplace=True)
    student_list.groupby('年级').get_group(name).head().to_excel(grade_excel, sheet_name=name+'-倒数前五')
grade_excel.save()
```

导出的效果还是不错的。

![](https://upload-images.jianshu.io/upload_images/12875160-573219dd26f62e61.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


#### 3 班级分析

##### 3.1 班级整体情况分析

分析各班第1在年级的位置，能够帮助学校在整体层面把握各班的教学质量。在班级排名的获取方法与获取年级排名的方法一致，对数据表“总分”这一列用“班级”这一列取groupby，然后对每个groupby进行rank()计算。

```python
student_list['班级排名'] = student_list['总分'].groupby(student_list['班级']).rank(ascending=False).astype(int)
student_list[student_list['班级排名'] == 1].groupby('年级').get_group('高三')
```
从下表可以看出，有个班级有两位同学拿了年级前10，真是天选之班。

![](https://upload-images.jianshu.io/upload_images/12875160-24e09bc682827acc.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

使用如下代码，进一步分析是哪个班。

```python
student_list.sort_values(['年级排名'],ascending=True,inplace=True)
student_list.groupby('年级').get_group('高三').head(10).groupby(['班级'])['年级排名'].count()
```
原来天选之班不仅仅只有一个……

![](https://upload-images.jianshu.io/upload_images/12875160-785d0c5972244074.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


##### 3.2 各班一本率及学科成绩分析

一本率可以说是高中班级中的重要数据了，以广西2019年高考理科一本线（509分）为标准，计算各班一本率。在对各班进行groupby的基础上，使用一个apply函数计算各班过一本线的人数，进而计算一本率。与年级整体数据的分析方法类似，按班级分析各科成绩的最高分、最低分、平均分、中位数，使用.merge()函数（相当于Excel中的vlookup函数）将各班各科的数据与前面的一本率数据匹配，最后按照一本线降序呈现数据。


```python
class_list = pd.DataFrame()
class_list['一本人数'] = student_list.groupby('班级')['总分'].apply(lambda x: np.sum((x >=510).astype(int)))
class_list['班级人数'] = student_num
class_list['一本率'] = class_list['一本人数']/class_list['班级人数']
subject_name = ['语文','数学','英语','物理','化学','生物','总分','年级排名']
class_list = class_list.merge(student_list.groupby('班级')[subject_name].agg(['max','min','mean','median']).reset_index(), on='班级', how='left')
class_list.head()
```
从图中可以看出，全校一本率最高的班级是1704班，但是一本率仅有36.2%，看来这个虚构的学校成绩并不好……

![](https://upload-images.jianshu.io/upload_images/12875160-c35b1919584b203e.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

##### 3.3 数据保存
按照类似与年级数据的保存方法，保存班级分析的数据。

```python
class_excel = pd.ExcelWriter(r'班级分析.xlsx')
class_list.to_excel(class_excel, sheet_name='整体情况')
for name in grade_name:
    student_list[student_list['班级排名'] == 1].groupby('年级').get_group(name).to_excel(class_excel, sheet_name=name)
class_excel.save()
```

![](https://upload-images.jianshu.io/upload_images/12875160-65c87176f65a576c.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


 #### 四、学科分析

##### 4.1 分数段分析
对于各个学科，比较重要的数据是各个年级、各个分数段的人数情况。使用的方法pd.cut()函数，对各个学科的各个分数段进行切割，并且保存为DataFrame（本文所有数据均为此格式）。由于在这个分析中有按照年级的循环求解，所以直接在开头就定义写入excel的subject_excel对象，在循环结尾直接.to_excel()保存。

```python
subject_excel = pd.ExcelWriter(r'学科分析.xlsx')

bins = [0,40,60,80,100,120,140,150]

group_name = ['高一','高二','高三']
subject_name = ['语文','数学','英语','物理','化学','生物']
for name in group_name:
    grade = student_list.groupby('年级').get_group(name)
    df = pd.DataFrame()
    for s_name in subject_name:
        cuts = pd.cut(grade[s_name],bins=bins) #可选label添加自定义标签
        subject_cut = grade.groupby(cuts)[s_name].count()
        df[s_name] = subject_cut
    df.index.name = name
    df.to_excel(subject_excel, sheet_name=name+'成绩分布')
df
```

从成绩区间可以看出，成绩主要分布在平均数附近，是很符合正态分布了。想起在前面分析年级倒数的时候有个学生生物81分，但是年级排名倒数第3，是很偏科了。

![](https://upload-images.jianshu.io/upload_images/12875160-a99de5b8eec31d7b.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

##### 4.2 偏科学生分析

偏科学生在某些学科是潜力股，发现偏科学生在某种意义上是发现“好苗子”的过程。在本次成绩表中，假定数学成绩大于130且语文和英语成绩小于90分的学生为偏科学生。先构建一个'数学偏科'的flag，然后进行筛选。在这一部分的最后，把学科分析部分的数据保存到excel中。

```python
student_list['数学偏科'] = ((student_list['数学']>=130) & (student_list['语文']<90) & (student_list['英语']<90)).astype(int)
partial = student_list[student_list['数学偏科'] == 1]
partial.drop(['数学偏科'],axis=1,inplace=True)
partial.to_excel(subject_excel, sheet_name='偏科学生')
subject_excel.save()
partial.head()
```
图中的晋启同学数学考了149，其他学科都不超过80，是很偏科了。

![](https://upload-images.jianshu.io/upload_images/12875160-f54e856ed61e7bab.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

使用如下代码，看看各个年级有多少人是数学偏科的。可以看到，高二年级的偏科人数最多。

```python
partial.groupby('年级')['姓名'].count()
```

![](https://upload-images.jianshu.io/upload_images/12875160-e8ba73ecbbdf8360.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

#### 五、思考

代码和文章写到这里已经写了 6个多小时了，需要写一个结尾总结一下收获，减轻一下不务正业，没有读文献的负罪感。有人说这些事情用Excel完全可以办到，为什么要在这里写代码呢？全文中使用到的方法，函数均为我在做一个大数据比赛时学到的函数，换一个场景使用证明我学会了；其次，那个比赛需要处理的数据量是1.47亿条，用Excel打不开（T_T），所以写代码也是适用于大规模的数据处理；另外，代码具有复用性，只要使用场景不变，数据格式不变，可以说是一劳永逸的，适合于重复机械的工作场景。比如高中学生考试，有周测、月考、段考、期末考、摸底考……，而跑一次代码就可以生成成绩报告，为啥还要每次重复操作Excel呢？

![](https://upload-images.jianshu.io/upload_images/12875160-ca3d012d3a3fd43c.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

#### 附录

这篇文章主要用到如下方法：

> 1. .groupby()系列方法，包括.groupby().agg(), .groupby().apply(), .groupby().get_group()等，用以实现分年级、分班级计算；
> 2. pd.read_excel(), pd.to_excel()，Excel的读取与存储；
> 3. pd.merge()，数据筛选，相当于Excel中的vlookup；
> 4. df.sort_values()，对数据进行排序；
> 5. df.drop()，按条件去除某行或某列；
> 6. df.cut()，给定区间，分析数据所属区间；
> 7. df.count()，分析某列数据各个元素的值，结合.groupby可以实现Excel的数据透视表效果。

其他方法（未在此场景中应用）：

> 1. np.unique(), 去除numpy的重复值，df.drop_duplicates()，按条件去除DataFrame的重复值；
> 2. pd.to_datetime()，将其他类型的时间数据转换为时间戳，df.dt.total_seconds()，把时间数据转化为秒；
> 3. df.diff(1)，计算某两列的差值（一般是计算dt时间内变量的变化量）
> 4. df.shift()，向上或向下移动某列数据；
> 5. df.fillna()，缺失值填充；
> 6. .loc[]，按当前索引提取某行；.iloc[]，按数字提取某行；
> 7. .isin()，分析某列元素是否在另一数组中。

源码链接：https://github.com/Fire2341/Learning_Summary