- **问题：** 

- 在Ambari上观察，一个节点上的所有服务都离线了，但在后台查看服务进程都在

-  

- **问题原因**：

- 检查每个受影响的主机上的/var/log/ambari-agent/ambari-agent.log会显示以下错误： 

-  ```verilog
  WARNING 2015-09-14 17:06:52,332 [PythonExecutor.py](http://pythonexecutor.py/):149 - {'msg': 'Unable to read structured output from /var/lib/ambari-agent/data/structured-out-status.json'} 
  
  WARNING 2015-09-14 17:06:52,372 [PythonExecutor.py](http://pythonexecutor.py/):149 - {'msg': 'Unable to read structured output from /var/lib/ambari-agent/data/structured-out-status.json'} 
  
  WARNING 2015-09-14 17:06:52,380 [PythonExecutor.py](http://pythonexecutor.py/):149 - {'msg': 'Unable to read structured output from /var/lib/ambari-agent/data/structured-out-status.json'} 
  
  WARNING 2015-09-14 17:06:52,420 [PythonExecutor.py](http://pythonexecutor.py/):149 - {'msg': 'Unable to read structured output from /var/lib/ambari-agent/data/structured-out-status.json'} 
  
  WARNING 2015-09-14 17:06:52,428 [PythonExecutor.py](http://pythonexecutor.py/):149 - {'msg': 'Unable to read structured output from /var/lib/ambari-agent/data/structured-out-status.json'} 
   ```

-  This is a known bug, reported in Ambari upstream: 

- https://issues.apache.org/jira/browse/AMBARI-13034 

-  

- **解决方案/替代方法：** 

- 如果在群集上观察到这些错误和症状，请执行以下步骤：

- 1. 通过ssh登录受影响的节点。
  2. 找到并删除/var/lib/ambari-agent/data/structured-out-status.json文件。
  3. 重新启动ambari-agent：service ambari-agent restart
  4. 返回Ambari Web界面并在相应主机上启动受影响的服务。这些服务应该出现并保持在线状态。 

- 此问题目前将在未来的Ambari版本中解决。 

-  