# Python - decimal


## Introduction
前陣子，要把別人的 SQL 程式用 Python 重現  
發現，有一部份的資料算出來的結果都有些微的誤差，人工驗算後才發現是浮點數 (floating-point) 的問題

## decimal 
以下就是跟天真的我預想不一樣的例子
```python
f = 3.15
round(f, 1)
# 3.1
```

如果精度 (precision) 很重要，請使用內建的 `decimal` 這個模組
```python
from decimal import Decimal
from decimal import ROUND_HALF_UP, ROUND_HALF_EVEN, ROUND_UP 
# 沒錯，rounding 規則也有很多種，要自己挑選
# 我自己是使用 ROUND_HALF_EVEN

f = 3.15
Decimal(f)
# Decimal('3.149999999999999911182158029987476766109466552734375')

Decimal(str(f)).quantize(Decimal('.0'), ROUND_HALF_EVEN)
# 3.2
```


## References
* https://kirin.idv.tw/python-decimal-module-tutorial/
* https://docs.python.org/3/library/decimal.html

