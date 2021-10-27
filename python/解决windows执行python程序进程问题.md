# 解决windows执行python程序concurrent.futures.ProcessPoolExecutor报：No such option: --multiprocessing-fork

You need a call to `multiprocessing.freeze_support()` when packaging a Python script into an executable for use on Windows. This call should come just after `if __name__ == '__main__':` before actually calling `main()`

Namely, you will need to call `multiprocessing.freeze_support()` right after your call to main:

```py
import multiprocessing

if __name__ == '__main__':
    multiprocessing.freeze_support()
```

> 参考链接：https://stackoverflow.com/questions/13514031/py2exe-with-multiprocessing-fails-to-run-the-processes