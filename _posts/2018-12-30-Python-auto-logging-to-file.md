python的logging模块提供了一个handler，可以解决定期保存日志到文件中。

```python
import logging
from logging.handlers import TimedRotatingFileHandler
import time

# 获取logger，可指定logger名字
logger = logging.getLogger()
# 设置日志级别
logger.setLevel(logging.INFO)
# 生成handler对象，可指定日志生成周期
handler = TimedRotatingFileHandler("a.log", when='D', interval=1, encoding="utf-8")
handler.setLevel(logging.DEBUG)
# 设置日志信息格式
formatter = logging.Formatter('%(asctime)s - %(name)s - %(filename)s - %(lineno)d - %(levelname)s - %(message)s')
handler.setFormatter(formatter)
logger.addHandler(handler)

i = 0
while True:
    logger.info("msg {}".format(i))
    i += 1
    time.sleep(2)
```

还可以根据日志文件大小来保存：

```python
from logging.handlers import RotatingFileHandler
用法类似TimedRotatingFileHandler
```

