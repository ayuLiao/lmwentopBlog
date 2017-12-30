# python处理execl使用方式
使用openpyxl这个第三方库来处理execl

基础步骤

1.加载第三方库,load_workbook方法将xlsx加载进程序

```
from openpyxl import load_workbook
```

2.告诉程序xlsx文件的位置，使用load_wordbook方法

```
file_path = '文件的位置'---字符串类型
wb = load_workbook(file_path)
```

3.获取xlsx文件中的一个工作表
可以先查了这个xlsx文件中有什么工作表

```
wb.get_sheet_names()
```

然后再通过工作表名字来加载具体某个工作表进行操作

```
ws = wb['工作表名字']
```

4.两种方式获得工作表对应字段的值

```
# 名字，从1开始
ws['A1'].value
# 横纵坐标，从0开始
ws.cell(row=0,column=0).value
```

5.写出数据，同样两种方法

```
# 通过名字直接写
ws['A1'] = 新的值
#通过坐标写
ws.cell(row=0, column=0, value=新的值）
```

6.最后不要忘记保存，不然前面写入的新数据是不会存到xlsx文件中的

```
# 存数据时使用wb，而不是ws，同时不要忘记将文件的路径告诉这个方法
wb.save(file_path)
```

简单题目，使用上面提到的内容，读取下图的表，将A、B、C中的值累加斌赋值给D

![](media/15136959342596/15136965767459.jpg)



答案：

```
from openpyxl import load_workbook
file_path = '/Users/ayuliao/Documents/test1.xlsx'
wb = load_workbook(file_path)
name = wb.get_sheet_names()[0]
ws = wb[name]
for i in range(1,13):
    A = 'A'+str(i)
    B = 'B'+str(i)
    C = 'C'+str(i)
    D = 'D'+str(i)
    ws[D] = ws[A].value + ws[B].value + ws[C].value
    
wb.save(file_path)
```

通过上面的程序处理后的结果：
![](media/15136959342596/15136969504805.jpg)

如果我想要将A、B、C、D中的最大值转换为最小值，如何做？

```
from openpyxl import load_workbook
file_path = '/Users/ayuliao/Documents/test1.xlsx'
wb = load_workbook(file_path)
name = wb.get_sheet_names()[0]
ws = wb[name]
min = 10000
max = 0
max_i,max_j = 0,0
for i in range(1,13):
    for j in range(1,5):
        k = ws.cell(row=i, column=j).value
        if min>k:
            min=k
        if max<k:
            max=k
            max_i = i
            max_j = j
print(max_i)
print(max_j)
print(min)
ws.cell(row=max_i, column=max_j,value=min)
    
wb.save(file_path)
```

在使用openpyxl处理execl表时，比较恶心的问题就是execl中每个格子的格式，比如，execl中我们看到的是日期，但它实际的格式却是自定义或者常规
![](http://obfs4iize.bkt.clouddn.com/execl%E6%97%A5%E6%9C%9F.png)
![](http://obfs4iize.bkt.clouddn.com/execl%E6%97%A5%E6%9C%9F2.png)

此时通过openpyxl读出来就会是数字，这个数字是有规律的，它是1900/1/0到该日期间隔的天数，所以我们同样可以通过python来计算这一天与1900/1/1间隔多少（现实中没有1月0日的说法，所以python中只能获得1900/1/1日）

如果想直接通过openpyxl获取execl中的日期，需要将execl中格子的格式修改为“日期”格式



