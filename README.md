# ssbc-onekey
磁力链接一键部署（基于SSBC）
搭建
直接使用脚本搭建安装，记住服务器内存最好1g以上的
wget --no-check-certificate https://raw.githubusercontent.com/banwagong-news/scripts/master/ssbc-setup.sh && bash ssbc-setup.sh
之后会让你输入下面的信息
等待一段时间就会有数据了，但是注意服务器一定要是国外的服务器，为什么要使用国外的服务器呢大家应该都懂的。

数据库相关
脚本安装的mariadb默认是不允许其他机器登录的，所以如果你要使用本地的数据库连接工具连接这个mariadb的话就要开启mariadb的root远程连接了，还有就是默认是没有密码的，所以最好你设置一个root密码，首先设置root密码，输入
mysql_secure_installation
之后按照提示操作就好
root@bboysoul-centos ssbc# mysql_secure_installation 
NOTE: RUNNING ALL PARTS OF THIS SCRIPT IS RECOMMENDED FOR ALL MariaDB
SERVERS IN PRODUCTION USE!  PLEASE READ EACH STEP CAREFULLY!
In order to log into MariaDB to secure it, we'll need the current
password for the root user.  If you've just installed MariaDB, and
you haven't set the root password yet, the password will be blank,
so you should just press enter here.
Enter current password for root (enter for none): 
OK, successfully used password, moving on...
Setting the root password ensures that nobody can log into the MariaDB
root user without the proper authorisation.
Set root password? Y/n y
New password: 
Re-enter new password: 
Password updated successfully!
Reloading privilege tables..
 ... Success!
By default, a MariaDB installation has an anonymous user, allowing anyone
to log into MariaDB without having to have a user account created for
them.  This is intended only for testing, and to make the installation
go a bit smoother.  You should remove them before moving into a
production environment.
Remove anonymous users? Y/n y
 ... Success!
Normally, root should only be allowed to connect from 'localhost'.  This
ensures that someone cannot guess at the root password from the network.
Disallow root login remotely? Y/n n
 ... skipping.
By default, MariaDB comes with a database named 'test' that anyone can
access.  This is also intended only for testing, and should be removed
before moving into a production environment.
Remove test database and access to it? Y/n y
Dropping test database...
... Success!
Removing privileges on test database...
... Success!
Reloading the privilege tables will ensure that all changes made so far
will take effect immediately.
Reload privilege tables now? Y/n y
 ... Success!
Cleaning up...
All done!  If you've completed all of the above steps, your MariaDB
installation should now be secure.
Thanks for using MariaDB!
之后就是开启mariadb的远程访问
首先登陆mariadb
mysql -u root -p
之后输入下面命令
MariaDB mysql> use mysql
Database changed
MariaDB mysql> update user set Host='%' where Host='localhost';
Query OK, 1 row affected (0.00 sec)
Rows matched: 1  Changed: 1  Warnings: 0
MariaDB mysql> flush privileges;
Query OK, 0 rows affected (0.00 sec)
MariaDB mysql> 
接着就可以远程登陆数据库了
之后要修改手撕包菜程序里面的连接密码
首先关闭相关的进程
ps -ef |grep python
一般就是下面几个进程
root       958     1  0 20:51 ?        00:00:00 /usr/bin/python -Es /usr/sbin/tuned -l -P
root      3604     1  0 21:13 pts/0    00:00:00 /usr/bin/python2 /usr/bin/gunicorn ssbc.wsgi:application -b 127.0.0.1:8000 --reload
root      3616  3604  1 21:13 pts/0    00:00:09 /usr/bin/python2 /usr/bin/gunicorn ssbc.wsgi:application -b 127.0.0.1:8000 --reload
root      3693     1 12 21:15 ?        00:01:30 python simdht_worker.py
root      3694     1  0 21:15 ?        00:00:00 python index_worker.py
kill之后再kill下面几个进程
ps -ef |grep search
root      3467     1  0 21:03 ?        00:00:00 searchd --config ./sphinx.conf
root      3468  3467  0 21:03 ?        00:00:02 searchd --config ./sphinx.conf
接着修改配置文件
vim /root/ssbc/sphinx.conf
增加数据库的密码
    sql_host                = 127.0.0.1
    sql_user                = root
    sql_pass                = 
    sql_db                  = ssbc
    sql_port                = 3306  # optional, default is 3306
vi /root/ssbc/workers/index_worker.py

SRC_HOST = '127.0.0.1'
SRC_USER = 'root'
SRC_PASS = ''
DST_HOST = '127.0.0.1'
DST_USER = 'root'
DST_PASS = ''
上面两个密码都要修改
vi /root/ssbc/workers/simdht_worker.py

DB_HOST = '127.0.0.1'
DB_USER = 'root'
DB_PORT = 3306
DB_PASS = ''
DB_NAME = 'ssbc'
BLACK_FILE = 'black_list.txt'

vim /root/ssbc/ssbc/settings.py
修改下面，root后面加上数据库密码
DATABASES = {

'default': {
    'ENGINE': 'django.db.backends.mysql',
    'NAME': 'ssbc',
    'HOST': '127.0.0.1',
    'PORT': 3306,
    'USER': 'root',
    'PASSWORD': 'magrco.com',
    'OPTIONS': {
       "init_command": "SET storage_engine=MYISAM",
    }
}
}

关于数据迁移
这个其实好办先在新的机器上执行脚本，执行完成之后删除数据库建立新的ssbc数据库记住编码要utf-8的，之后把老的数据库导入新的就可以了
其他的使用技巧

1.必须centos7吗？
非常建议使用centos7，centos6可能会有意想不到的错误
2.如何设置首页关键字？
登录管理员后台，点击Rec keywordss，右上角新增
3.怎么查看入库的文件？
登录管理员后台，点击 Hashs 
4.怎么查看每天入库了多少文件，以便清楚入库效率？
登录管理员后台，点击 Status reports 
5.如何确认web服务器、采集、入库正在运行？
运行 ps -ef|grep python|grep -v grep
结果里面有
gunicorn ssbc.wsgi:application -b 127.0.0.1:8000 --reload 
python simdht_worker.py  
python index_worker.py
即表示正在运行。
——————————————————————————————————————
去除搜索页 右下角广告
root@localhost ssbc-master# cd web/static/js
root@localhost js# vi ssbc.js   找到如下3行，在前面添加//进行注释，保存
//        document.write('<script src="http://v.6dvip.com/ge/?s=47688"><\/script>');
//            document.writeln("<script language=\"JavaScript\" type=\"text/javascript\" src=\"http://js.6dad.com/js/xiaoxia.js\"></script>");
//           document.writeln("<script language=\"JavaScript\" type=\"text/javascript\" src=\"http://js.ta80.com/js/12115.js\"></script>");
—————————————————————————————————————
如何修改扩展名归类？
workers/metautils.py文件中有如下代码：
def get_category(ext):
ext = ext + '.'
cats = {
    u'video': '.avi.mp4.rmvb.m2ts.wmv.mkv.flv.qmv.rm.mov.vob.asf.3gp.mpg.mpeg.m4v.f4v.',
    u'image': '.jpg.bmp.jpeg.png.gif.tiff.',
    u'document': '.pdf.isz.chm.txt.epub.bc!.doc.ppt.',
    u'music': '.mp3.ape.wav.dts.mdf.flac.',
    u'package': '.zip.rar.7z.tar.gz.iso.dmg.pkg.',
    u'software': '.exe.app.msi.apk.'
}
意思是：扩展名为.exe、.app、.msi、,.apk的文件都属于software类型。
如果你把u'software': '.exe.app.msi.apk.' 改为 u'software': 'app.msi.apk.'，那么exe将会被归为other类型。
所以在这里修改归类设置。
——————————————————————————————————————
如何禁止某些格式/分类的文件入库？
workers/metadata.py文件中有如下代码：
info'extension' = metautils.get_extension(bigfname).lower()
info'category' = metautils.get_category(info'extension')
所以如果你要排除扩展名为.exe的文件，或者类型为software，可以在上面代码后面加上
##########这是增加的过滤-开始############
#按扩张名过滤，禁止扩展名为.exe的入库
if info'extension' == 'exe':
return # 直接返回，跳过下面的入库
#按文件类型过滤，禁止类型为software的入库
if info'category' == 'software':
return
#禁止类型为other的入库
if info'category' == 'other':
return

——————————————————————————————————————
如何重建索引？
第一步：
删除/data目录
第二步：
进入数据库，把search_hash表中所有记录的tagged字段置为0。
UPDATE search_hash SET tagged=0
然后启动sphinx、index_worker.py。
——————————————————————————————————————
MySQL server has gone away提示怎么办？
ssbc 运行一段时间后，大概半个小时，就莫名奇妙停止不爬了。 错误提示如下：
MySQL server has gone away
通过错误提示可以看出，其实是ssbc与mysql(maridb)断开连接了，导致程序异常，当然就插入不了数据了。
有3种解决办法：
方法1是写个脚本，定时重启爬虫。
方法2是修改下代码，当mysql断开连接时，再次重连mysql就可以拉。
方法3是修改Mysql配置，将闲置时间wait_timeout设置长一点。
——————————————————————————————————————
哪里设置爬虫线程？让爬虫爬快/慢点？
在workers/simdht_worker.py里面把MAX_QUEUE_LT、MAX_QUEUE_PT、max_node_qsize设大/小一点。
如何关闭调试模式？设置404页面？
——————————————————————————————————————
如何在搜索结果页面添加迅雷链接？
在web/views.py文件加入以下代码生成迅雷链接:
import base64
xunleiurl = 'AAmagnet:?xt=urn:btih:' + d'info' + 'ZZ'
d'xunlei_url' = 'thunder://' + base64.b64encode(xunleiurl)
可以在模板中用“ {{xunlei_url}} ”调用。位置要放在return render(request, 'info.html', d)的前面。
——————————————————————————————————————
SSBC如何搬家？
数据库用mysqldump导出sql，在新服务器上运行一键包，再导入刚才的sql。
——————————————————————————————————————
提示duplicate id 'xxxx'解决办法
进入数据库，执行语句
update search_hash set tagged=True where id=xxxx;
——————————————————————————————————————
搜索中文报错
解决办法：
如果是centos7系统，修改/usr/lib64/python2.7/site.py
vi  /usr/lib64/python2.7/site.py
在import sys下添加2行：
reload(sys)
sys.setdefaultencoding('utf8')
——————————————————————————————————————
如何关闭调试模式？设置404页面
ssbc 是 基于 django 框架写的如何关闭调试模式呢？
在 ssbc/setting.py 中将 DEBUG = True 改为 DEBUG = False
这样就关闭了调试模式，但还没完。现在访问，会直接出现 500 的错误，这是静态资源造成的
下面给出解决方案
在 ssbc/setting.py 中将
STATIC_ROOT = os.path.join(BASE_DIR, 'www/static')
修改为
STATIC_ROOT = os.path.join(BASE_DIR, 'web/static')
将下面这行注释掉
STATICFILES_STORAGE = 'django.contrib.staticfiles.storage.ManifestStaticFilesStorage'
在 ssbc/urls.py 中导入 settings 包，然后再 urlpatterns 中添加
url(r'^static/(?P<path>.*)$','django.views.static.serve',{'document_root':settings.STATIC_ROOT})
现在访问出现404后，显示的是我们制定的404页面，而不是错误页面了。

注意：在修改正在运行的程序是一定要结束运行再修改。
更多信息请访问：http://wiz.im  http://magrco.com
