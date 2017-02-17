#安装使用
根据官网https://developer.android.com/studio/profile/systrace.html的说明
mac电脑的路径是/Users/xinmin/Library/Android/sdk/platform-tools/systrace，
然后运行命令./systrace.py --time=10 -o mynewtrace.html sched gfx view wm 的时候发现下面的错误
```
Traceback (most recent call last):
  File "./systrace.py", line 48, in <module>
    from systrace import run_systrace
  File "/Users/xinmin/Library/Android/sdk/platform-tools/systrace/catapult/systrace/systrace/run_systrace.py", line 43, in <module>
    from systrace import systrace_runner
  File "/Users/xinmin/Library/Android/sdk/platform-tools/systrace/catapult/systrace/systrace/systrace_runner.py", line 15, in <module>
    from systrace.tracing_agents import battor_trace_agent
  File "/Users/xinmin/Library/Android/sdk/platform-tools/systrace/catapult/systrace/systrace/tracing_agents/battor_trace_agent.py", line 11, in <module>
    from battor import battor_wrapper
  File "/Users/xinmin/Library/Android/sdk/platform-tools/systrace/catapult/common/battor/battor/battor_wrapper.py", line 22, in <module>
    import serial
ImportError: No module named serial
```
问题出在pyserial库上
解决方案是到到https://pypi.python.org/pypi/pyserial#downloads 下载pyserial到目录下/Users/xinmin/Downloads/pyserial-3.2.1 
并执行命令sudo python setup.py install就可以使用了
