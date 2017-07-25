Overlap Studies Functions（重叠研究函数）
#### BBANDS-布林带<br> 
```python
upperband, middleband, lowerband = BBANDS(close, timeperiod=5, nbdevup=2, nbdevdn=2, matype=0)
```
#### DEMA-双指数移动平均<br>
real = DEMA(close, timeperiod=30)

#### EMA-指数移动平均<br>
注意：EMA函数具有不稳定的周期<br>
real = EMA(close, timeperiod=30)
