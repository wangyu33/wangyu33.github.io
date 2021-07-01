# Pandas常用操作

## pd.read_csv()读入文件

header=None代表无表头



## pd.to_csv()写入文件

df.to_csv(path, mode = , header = , index= )

mode用于判断写入类型，'a'为追加写入，’w‘为覆盖写入

header为列名，index为索引列



## pd 切片索引

1. 只取行的时候，不对列进行筛选

1.1 第一种情况是只取某一行。用df.iloc[行号]，也可以直接df.iloc[[行号]]。前者是个series；后者是个df；但不能直接df[行号],df[]里如果要直接引用，只能是列名。

```text
testdf3.iloc[0] # pandas series
testdf3.iloc[[0]] # dataframe
```

1.2 第二种情况是取连续的某几行。用df.iloc[行号：行号]，也可以偷懒用df[行号：行号]。得到的都是df.

```text
testdf3[2:3] # data frame
testdf3.iloc[2:3] # data frame
```

1.3 第三种情况是不连续的多行，则是df.iloc[[行号，行号]],注意是两个方括号。得到的是df。

```text
testdf3.iloc[[1,3]]
```

2. 只取列的时候，不对行做筛选

2.1 只取一列，可以偷懒不用.loc

```text
testdf3['A'] # 单独一列是个series
testdf3.loc[:,'A'] # 同上，但比较复杂，一般不用
testdf3.iloc[:,0] # 同上，可以在不知道列名的时候用

testdf3[['A']] #单独一列是个df
testdf3.loc[:,['A']] # 同上，但比较复杂，一般不用
testdf3.iloc[:,[0]] # 同上，可以在不知道列名的时候用
```



2.2 取指定的某几列，可以偷懒不用.loc

```text
testdf3[['A','C']] # DF, 指定某几列，直接用列名
testdf3.loc[:,['A','C']] #  同上，但比较复杂，一般不用
testdf3.iloc[:,[0,2]] # 同上，可以在不知道列名的时候用 
```



2.3 取指定的连续几列，不能偷懒了，必须用.loc

```text
testdf3.loc[:,'A':'D'] #指定连续列，用列名 
testdf3.iloc[:,0:4] # 指定连续列，用数字
```

3. 取行的同时，也取列。一个原则是行偷懒的方式和列偷懒的方式都不能用了。必须用.loc或.iloc。

第一种情况是列索引用数字表示, df.iloc[行索引表达，列索引表达]，规则跟上面行索引一模一样。

```text
testdf3.iloc[[1,3],[0]] # dataframe
testdf3.iloc[[1,3],0] # series
testdf3.iloc[[1,3],1:3] # dataframe
testdf3.iloc[[1,3],[1,3]]
```

第二种情况是列索引直接引列名（行索引不存在这个问题，因为pandas没有所谓'行名'），就要用df.loc[行索引，列名索引。

```text
testdf3.loc[1,["A","D"]] # series 对应上述1.1
testdf3.loc[[1],["A","D"]] # df 对应上述1.1
testdf3.loc[[1,3],"A":"D"] # df 对应上述1.2
testdf3.loc[[1,3],["A","D"]] # df 对应上述1.3
```

## pd.groupby()操作

```python
DataFrame.groupby(by=None, axis=0, level=None, as_index=True, sort=True, group_keys=True, squeeze=False, observed=False, **kwargs)
```

by 为所用到的column, 但其实也可以是与df同行的Series.
as_index 是指是否将groupby的column作为index, 默认是True



## pd.apply()操作

df['col'].apply(lambda x: gao(x))





