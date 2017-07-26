TA-Lib中文手册
====
安装和故障排查
----
### 安装
你可以通过PyPI安装：
```
$ pip install TA-Lib
```
#### 排除安装错误
```
func.c:256:28: fatal error: ta-lib/ta_libc.h: No such file or directory compilation terminated.
```
如果你得到类似上图的错误，这通常意味着它找不到底层`TA-Lib`库，需要安装。
### 依赖
为了python能使用TA-Lib，你需要确保已经安装了[TA-Lib](http://ta-lib.org/hdr_dw.html)库。
#### Mac OS X
```
$ brew install ta-lib
```
#### Windows
下载[ta-lib-0.4.0-msvc.zip](https://sourceforge.net/projects/ta-lib/files/ta-lib/0.4.0/ta-lib-0.4.0-msvc.zip/download?use_mirror=nchc)并且解压到`C:\ta-lib`
#### Linux
下载[ta-lib-0.4.0-src.tar.gz](https://sourceforge.net/projects/ta-lib/files/ta-lib/0.4.0/ta-lib-0.4.0-src.tar.gz/download?use_mirror=nchc)并且：
```
$ untar and cd
$ ./configure --prefix=/usr
$ make
$ sudo make install
```
>如果你用`make -jX`来安装`TA-Lib`将会失败，但是没关系，直接在`[sudo] make install`后面重新运行`make -jX`就行了。

函数的API例子
----
类似于TA-Lib库，函数接口对公开的talib指标提供了一个轻量级的封装器。每个函数返回一个输出的数组，并为它们的参数设置默认值，除非指定为关键字参数。通常情况下，这些函数将有一个初始的“回望”时期（产生一个输出之前所需要的观测数目）被设置为空值`NaN`。

所有下面的例子都使用函数的API接口：
```python
import numpy
import talib

close = numpy.random.random(100)
```
计算收盘价的简单移动平均：
```python
output = talib.SMA(close)
```
计算布林线，使用三重指数移动平均：
```python
from talib import MA_Type

upper, middle, lower = talib.BBANDS(close, matype=MA_Type.T3)
```
计算收盘价的动量，时间周期为5：
```python
output = talib.MOM(close, timeperiod=5)
```

抽象API快速上手
----
如果您已经熟悉使用函数API，那么您就应该很容易使用抽象API。每个函数的输入都相同，都是numpy的arrays构成的字典。
```python
import numpy as np
# note that all ndarrays must be the same length!
inputs = {
    'open': np.random.random(100),
    'high': np.random.random(100),
    'low': np.random.random(100),
    'close': np.random.random(100),
    'volume': np.random.random(100)
}
```
函数可以直接导入，也可以用名称实例化：
```python
from talib import abstract
sma = abstract.SMA
sma = abstract.Function('sma')
```

在抽象API里，调用函数基本上与函数API相同：
```python
from talib.abstract import *
output = SMA(input_arrays, timeperiod=25) # calculate on close prices by default
output = SMA(input_arrays, timeperiod=25, price='open') # calculate on opens
upper, middle, lower = BBANDS(input_arrays, 20, 2, 2)
slowk, slowd = STOCH(input_arrays, 5, 3, 0, 3, 0) # uses high, low, close by default
slowk, slowd = STOCH(input_arrays, 5, 3, 0, 3, 0, prices=['high', 'low', 'open'])
```

高级用法
----
对于TA-Lib库更高级的使用案例，抽象API提供了更大的灵活性。你甚至可以自己定义`abstract.Function`的子类或者重写函数接受的输入数据的类型（比如pandas的DataFrame类型）

有关每个功能的详细信息可以通过信息属性访问：
```python
print Function('stoch').info
{
  'name': 'STOCH',
  'display_name': 'Stochastic',
  'group': 'Momentum Indicators',
  'input_names': OrderedDict([
    ('prices', ['high', 'low', 'close']),
  ]),
  'parameters': OrderedDict([
    ('fastk_period', 5),
    ('slowk_period', 3),
    ('slowk_matype', 0),
    ('slowd_period', 3),
    ('slowd_matype', 0),
  ]),
  'output_names': ['slowk', 'slowd'],
}
```

或者是人类可读的格式：
```python
help(STOCH)
str(STOCH)
```

其他实用的`Function`的属性：
```python
Function('x').function_flags
Function('x').input_names
Function('x').input_arrays
Function('x').parameters
Function('x').lookback
Function('x').output_names
Function('x').output_flags
Function('x').outputs
```

除了直接调用函数，函数在被设置之后会保持状态并且记住他们的参数/输入的arrays.你可以设置参数并且使用新的输入数据通过run()来重新计算：
```python
SMA.parameters = {'timeperiod': 15}
result1 = SMA.run(input_arrays1)
result2 = SMA.run(input_arrays2)

# Or set input_arrays and change the parameters:
SMA.input_arrays = input_arrays1
ma10 = SMA(timeperiod=10)
ma20 = SMA(20)
```

更多细节，请查看[代码](https://github.com/mrjbq7/ta-lib/blob/master/talib/abstract.pyx#L46)

所有支持的指标和函数
----
### Overlap Studies Functions（重叠研究函数）

#### BBANDS-布林带
```python
upperband, middleband, lowerband = BBANDS(close, timeperiod=5, nbdevup=2, nbdevdn=2, matype=0)
```
了解更多关于布林带请点击[tadoc.org](http://www.tadoc.org/indicator/BBANDS.htm)

#### DEMA-双指数移动平均
```python
real = DEMA(close, timeperiod=30)
```
了解更多关于双指数移动平均请点击[tadoc.org](http://www.tadoc.org/indicator/DEMA.htm)

#### EMA-指数移动平均
注意：`EMA`函数具有不稳定的周期
```python
real = EMA(close, timeperiod=30)
```
了解更多关于指数移动平均请点击[tadoc.org](http://www.tadoc.org/indicator/EMA.htm)


