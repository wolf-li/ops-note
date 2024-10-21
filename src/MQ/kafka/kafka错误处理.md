1. 问题：Error: JMX connector server communication error: service: sun.management.AgentConfigurationError: java.rmi.server.ExportException: Port already in use: 9999; nested exception is
解决方法：
1. 查找并杀死占用端口的进程：
使用以下命令找到占用端口 9999 的进程，并终止它：
```bash
Copy code
sudo lsof -i :9999
sudo kill -9 <PID>
```
请注意， <PID> 是你找到的占用端口的进程的 PID。
2. 修改 bin/kafka-run-class.sh JMX_PORT 端口