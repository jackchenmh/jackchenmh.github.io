远程链接Ipython Notebook配置
====
    最近经常跑数据分析,因为习惯本地跑ipython notebook分析,但数据都在server上,每次scp很麻烦,找了一下远程连接到服务器上的ipython notebook方法.
    这里有一份文档,但是版本比较旧,我按照现在ipython的版本配置了一下,主要参考是这个.
    
    * 首先在远程server上,安装ipython

    * 随便一个server上的位置,输入ipython,然后按照以下方法得到一个哈希的密码,保存起来 
        from notebook.auth import passwd 
        passwd() 

    * 到目录~/.jupyter/下,创建默认的jupyter_notebook_config.py,如果没有默认创建,直接vim即可. 然后在文件内输入

        c = get_config()
        c.NotebookApp.ip='*'
        c.NotebookApp.password = u'...' # 这里输入第二步生成的哈希密码
        c.NotebookApp.open_browser = False
       c.NotebookApp.port = 1224 # 猜猜为什么是这个号码 :)
    * 远程的配置完成了,下面移到远程server要访问的文件夹,输入ipython notebook --no-browser --port=1224
    * server端的ipython以及启动,下面就是本地连接. 输入 ssh -N -f -L 127.0.0.1:1314:127.0.0.1:1224 user@address,
      这是说把本地的1314端口连接到server上1224的端口.
    * 随便一个浏览器,地址栏localhost:1314,然后就会看到一个输入密码.(我会说我一直用123123123?)然后大功告成.
    * 还有一种更加直接的方法,在第3步之后,既然server上的Ipython Notebook已经启动了,那可以直接在浏览器打开address:1224的方法,
      一样的界面
