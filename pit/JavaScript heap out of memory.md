# 报错 JavaScript heap out of memory
 安装increase-memory-limit
 ```shell
 npm i -g increase-memory-limit
 # in project folder
 increase-memory-limit
 ```
# 可能还有的错误
## 系统上禁止运行脚本 increase-memory-limit
```shell
increase-memory-limit : 
无法加载文件 C:\Users\MSI\AppData\Roaming\npm\increase-memory-limit.ps1，
因为在此系统上禁止运行脚本。有关详细信息，
请参阅 https:/go.microsoft.com/fwlink/?LinkID=135170 中的 about_Execution_Policies。
```

核心是power shell的安全策略，将 nrm 命令视为了不安全脚本，不允许执行。只需要放开权限就行。

我们通过管理员权限运行power shell，然后输入命令

```shell
set-ExecutionPolicy RemoteSigned
```
选择“是”，就OK了。

## ‘“node --max-old-space-size=10240“‘不是内部或外部命令，也不是可运行的程序

而是因为win10系统命令行中不能正确识别双引号""，所以要把这个插件包中涉及到的脚本中双引号都去掉，即修改node_modules下的.bin文件中的所有.cmd文件，将里面的"%_prog%" 去掉双引号 改成 %_prog%。

在ide界面bin目录上查找"%_prog%" 替换即可

