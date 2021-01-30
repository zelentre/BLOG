# 关于修改MYSQL数据库存储文件路径，MYSQL无法启动的解决方案

<!-- more -->

在修改MYSQL数据库存储文件路径期间，出现了一个问题：路径修改完，MYSQL再也启动不了。

![在这里插入图片描述](https://img-blog.csdnimg.cn/20200531095408765.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzM5OTEyNzgx,size_16,color_FFFFFF,t_70)
解决方案：
1、检查你更改路径的my.ini文件。****注意你的新路径使用的是“\”而不是“/”\*！！！！！！！！！！！***
如图所示：
![红色为原路径、绿色为新路径](https://img-blog.csdnimg.cn/20200531095850291.png)
图中：红色为原路径、绿色为新路径。再说一次****注意你的新路径使用的是“\”而不是“/”\*！！！！！！！！！！！***

2、如果经过上述操作，还不能重启成功。那么继续：
右击 ” 我的电脑 ” 找到“ 管理 ”：
![在这里插入图片描述](https://img-blog.csdnimg.cn/20200531100116741.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzM5OTEyNzgx,size_16,color_FFFFFF,t_70)
点击 “工具” 找到 “计算机管理” 点击，找到 ”本地用户和组“
![在这里插入图片描述](https://img-blog.csdnimg.cn/20200531100143602.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzM5OTEyNzgx,size_16,color_FFFFFF,t_70)
选择“组”–>双击Administrators–>单击“添加”–>单击“高级”–>单击“立即查找”–>在下面的列表中选择Network Service用户–>两次单击“确定”–>加入。
![在这里插入图片描述](https://img-blog.csdnimg.cn/20200531100207233.png)
好了，然后在重启mysql服务就没问题了，MYSQL数据库存储路径就修改好了，在新建个数据库，就会存到我们修改的路径下了