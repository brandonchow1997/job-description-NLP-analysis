
# 读取数据


```python
import pymongo
import pandas as pd
from pandas import Series
client = pymongo.MongoClient('localhost',27017)
db  = client['Graduation_project']
table = db['jobs_desc_data']
data = pd.DataFrame(list(table.find()))
del data['_id']
del data['省']
```


```python
data.head()
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>公司名称</th>
      <th>公司规模</th>
      <th>城市</th>
      <th>学历要求</th>
      <th>工作经验</th>
      <th>职位名称</th>
      <th>职位描述</th>
      <th>职位薪资</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>车主邦</td>
      <td>100-499人</td>
      <td>北京</td>
      <td>本科</td>
      <td>3-5年</td>
      <td>数据分析</td>
      <td>职位描述：1、熟悉公司运作，对各部门的数据统计分析工作给予支持配合。推送数据报表，为业务提供...</td>
      <td>20k-23k</td>
    </tr>
    <tr>
      <th>1</th>
      <td>一指遥</td>
      <td>100-499人</td>
      <td>北京</td>
      <td>本科</td>
      <td>1-3年</td>
      <td>数据分析师</td>
      <td>1.根据各部门数据需求，统计整理数据，并按业务部门需求形式呈现并反馈给业务部门；2.根据各部...</td>
      <td>4k-5k</td>
    </tr>
    <tr>
      <th>2</th>
      <td>宜信公司</td>
      <td>10000人以上</td>
      <td>北京</td>
      <td>本科</td>
      <td>经验不限</td>
      <td>数据分析专员</td>
      <td>职位说明负责日常的业绩报表制作、维护，以及业绩分析报告的撰写，并负责自动化报表的开发及测试工...</td>
      <td>10k-12k</td>
    </tr>
    <tr>
      <th>3</th>
      <td>会分期</td>
      <td>100-499人</td>
      <td>北京</td>
      <td>本科</td>
      <td>3-5年</td>
      <td>高级数据分析师</td>
      <td>数据分析主管岗位职责： 1. 通过数据的挖掘，密切关注运营进程，参与制定运营管理策略，提出改...</td>
      <td>15k-25k</td>
    </tr>
    <tr>
      <th>4</th>
      <td>滴滴出行</td>
      <td>10000人以上</td>
      <td>北京</td>
      <td>本科</td>
      <td>1-3年</td>
      <td>数据分析</td>
      <td>"工作职责：1.负责滴滴核心出行业务团队的数据分析、模型训练；2.利用技术实现业务的原始数据...</td>
      <td>25k-50k</td>
    </tr>
  </tbody>
</table>
</div>



### 输出到csv


```python
data.to_csv('jobs_with_description.csv')
```

___

# jieba实验

## 读取软技能dict


```python
with open('软技能.txt',"r") as f:
    soft_skills = f.read()
skilllist = soft_skills.split('\n')
```

## 读取硬技能dict


```python
with open('硬技能.txt',"r") as f:
    hard_skills = f.read()
```

## 读取停用词dict


```python
with open('Stopword.txt',"r",encoding='utf-8') as f:
    stopwords = f.read()
stopwords = stopwords.split('\n')
```

___

### 引入jieba分词库，并读取用户字典


```python
import jieba
jieba.load_userdict("userdict.txt")
```

---

### 列表初始化


```python
# 初始化软技能列表
soft_skills_count_list = []
# 初始化硬技能列表
hard_skills_count_list = []
```

## 软技能匹配函数


```python
def match_soft(skills):
    for item in skills:
        if item not in stopwords:
            if item in soft_skills:
                soft_skills_count_list.append(item)
```

### 匹配职位描述中的软技能


```python
for item in data['职位描述']:
    skill_list = jieba.lcut(item)
    #print(skill_list)
    skill_list = list(set(skill_list)) # list去重
    #print(skill_list)
    match_soft(skill_list)
```


```python
soft_skills = Series(soft_skills_count_list).value_counts()
soft_skills[:10]
```




    数据分析    2434
    运营      1334
    数据挖掘     888
    建模       824
    模型       768
    逻辑思维     716
    管理       710
    学习       676
    数据库      582
    算法       580
    dtype: int64



## 硬技能匹配函数


```python
def match_hard(skill_list):
    for skill in skill_list:
        # 单词转换为首字母大写，其余小写
        if skill.capitalize() not in stopwords:
            if skill.capitalize() in hard_skills:
                hard_skills_count_list.append(skill.capitalize())
```

### 正则提取英文单词


```python
import re
pattern = '[a-zA-Z]+'
#skill_list = re.findall(pattern,data['职位描述'])
```


```python
for item in data['职位描述']:
    skill_list = re.findall(pattern,item)
    skill_list = list(set(skill_list)) # list去重
    match_skills(skill_list)
```


```python
hard_skills = Series(hard_skills_count_list).value_counts()
hard_skills[:10]
```




    Sql        728
    Excel      581
    Python     532
    Sas        348
    Spss       345
    Hive       245
    Ppt        217
    Hadoop     188
    Spark      136
    Tableau    118
    dtype: int64


