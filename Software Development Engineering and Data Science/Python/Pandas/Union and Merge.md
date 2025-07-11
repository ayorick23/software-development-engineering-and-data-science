# Union and Merge


```python
import pandas as pd
```

## Example data


```python
#Datos de ejemplo
df1 = pd.DataFrame({"ID": [1, 2, 3], "Name": ["Dereck", "Ale", "Cris"], "Age": [25, 28, 42]})
df2 = pd.DataFrame({"ID": [1, 2, 3], "Name": ["Henry", "Guadalupe", "Eli"], "Age": [23, 27, 36]})
```

## Concatenating DataFrames (vertical join)


```python
#Concatenación de DataFrames (unión vertical)
print(pd.concat([df1, df2], ignore_index=True))
```

       ID       Name  Age
    0   1     Dereck   25
    1   2        Ale   28
    2   3       Cris   42
    3   1      Henry   23
    4   2  Guadalupe   27
    5   3        Eli   36
    

## Concatenating DataFrames (horizontal join)


```python
df3 = pd.DataFrame({"ID": [1, 2, 3], "Salary": [1000, 2000, 3000]})

#Concatenación de DataFrames (unión horizontal)
print(pd.concat([df1, df3], axis=1))
```

       ID    Name  Age  ID  Salary
    0   1  Dereck   25   1    1000
    1   2     Ale   28   2    2000
    2   3    Cris   42   3    3000
    

## Merge (join by common key, similar to JOIN in SQL)


```python
#Merge (combinación por clave común, similar a JOIN en SQL)
print(pd.merge(df1, df2, on="ID", how="inner"))
```

       ID  Name_x  Age_x     Name_y  Age_y
    0   1  Dereck     25      Henry     23
    1   2     Ale     28  Guadalupe     27
    2   3    Cris     42        Eli     36
    

## Merge with left join


```python
#Merge con left join
print(pd.merge(df1, df2, on="ID", how="left"))
```

       ID  Name_x  Age_x     Name_y  Age_y
    0   1  Dereck     25      Henry     23
    1   2     Ale     28  Guadalupe     27
    2   3    Cris     42        Eli     36
    

## Merge with right join


```python
#Merge con right join
print(pd.merge(df1, df2, on="ID", how="right"))
```

       ID  Name_x  Age_x     Name_y  Age_y
    0   1  Dereck     25      Henry     23
    1   2     Ale     28  Guadalupe     27
    2   3    Cris     42        Eli     36
    

## Merge with outer join


```python
#Merge con outer join
print(pd.merge(df1, df2, on="ID", how="outer"))
```

       ID  Name_x  Age_x     Name_y  Age_y
    0   1  Dereck     25      Henry     23
    1   2     Ale     28  Guadalupe     27
    2   3    Cris     42        Eli     36
    
