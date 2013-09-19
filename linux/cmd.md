# 移除apache2自启动脚本,
# 参数-f是为了解决 update-rc.d: /etc/init.d/apache2 exists during rc.d purge的问题
sudo update-rc.d -f apache2 remove
 
# 同时也可以方便的恢复自启动脚本
sudo update-rc.d apache2 defaults
 
# 或者编辑文件 /etc/default/apache2 设置内容中的 NO_BOOT 值为 1.
