# ssbc-onekey
磁力链接一键部署（基于SSBC）</br>
实例：HTTP://DHT.IM </br>
<h2>搭建</h2>
直接使用脚本搭建安装，记住服务器内存最好1g以上的</br>
bash---
wget --no-check-certificate https://raw.githubusercontent.com/magrco/ssbc-onekey/master/ssbc-onekey.sh && bash ssbc-onekey.sh ----
之后会让你输入域名，用户名，密码等信息</br>
等待一段时间就会有数据了，但是注意服务器一定要是国外的服务器，为什么要使用国外的服务器呢大家应该都懂的。</br>

<h2>数据库相关</h2></br>
脚本安装的mariadb默认是不允许其他机器登录的，所以如果你要使用本地的数据库连接工具连接这个mariadb的话就要开启mariadb的root远程连接了，还有就是默认是没有密码的，所以最好你设置一个root密码，首先设置root密码，输入</br>
mysql_secure_installation</br>
之后按照提示操作就好</br>
root@bboysoul-centos ssbc# mysql_secure_installation </br>
NOTE: RUNNING ALL PARTS OF THIS SCRIPT IS RECOMMENDED FOR ALL MariaDB</br>
SERVERS IN PRODUCTION USE!  PLEASE READ EACH STEP CAREFULLY!</br>
In order to log into MariaDB to secure it, we'll need the current</br>
password for the root user.  If you've just installed MariaDB, and</br>
you haven't set the root password yet, the password will be blank,</br>
so you should just press enter here.</br>
Enter current password for root (enter for none): </br>
OK, successfully used password, moving on...</br>
Setting the root password ensures that nobody can log into the MariaDB</br>
root user without the proper authorisation.</br>
Set root password? Y/n y </br>
New password: </br>
Re-enter new password: </br>
Password updated successfully!</br>
Reloading privilege tables..</br>
 ... Success!</br>
By default, a MariaDB installation has an anonymous user, allowing anyone</br>
to log into MariaDB without having to have a user account created for</br>
them.  This is intended only for testing, and to make the installation</br>
go a bit smoother.  You should remove them before moving into a</br>
production environment.</br>
Remove anonymous users? Y/n y</br>
 ... Success!</br>
Normally, root should only be allowed to connect from 'localhost'.  This</br>
ensures that someone cannot guess at the root password from the network.</br>
Disallow root login remotely? Y/n n</br>
 ... skipping.</br>
By default, MariaDB comes with a database named 'test' that anyone can</br>
access.  This is also intended only for testing, and should be removed</br>
before moving into a production environment.</br>
Remove test database and access to it? Y/n y</br>
Dropping test database...</br>
... Success!</br>
Removing privileges on test database...</br>
... Success!</br>
Reloading the privilege tables will ensure that all changes made so far</br>
will take effect immediately.</br>
Reload privilege tables now? Y/n y</br>
 ... Success!</br>
Cleaning up...</br>
All done!  If you've completed all of the above steps, your MariaDB</br>
installation should now be secure.</br>
Thanks for using MariaDB!</br>
之后就是开启mariadb的远程访问</br>
首先登陆mariadb</br>
mysql -u root -p</br>
之后输入下面命令</br>
MariaDB mysql> use mysql</br>
Database changed</br>
MariaDB mysql> update user set Host='%' where Host='localhost';</br>
Query OK, 1 row affected (0.00 sec)</br>
Rows matched: 1  Changed: 1  Warnings: 0</br>
MariaDB mysql> flush privileges;</br>
Query OK, 0 rows affected (0.00 sec)</br>
MariaDB mysql> </br>
接着就可以远程登陆数据库了</br>
之后要修改手撕包菜程序里面的连接密码</br>
首先关闭相关的进程</br>
ps -ef |grep python</br>
一般就是下面几个进程</br>
root       958     1  0 20:51 ?        00:00:00 /usr/bin/python -Es /usr/sbin/tuned -l -P</br>
root      3604     1  0 21:13 pts/0    00:00:00 /usr/bin/python2 /usr/bin/gunicorn ssbc.wsgi:application -b 127.0.0.1:8000 --reload</br>
root      3616  3604  1 21:13 pts/0    00:00:09 /usr/bin/python2 /usr/bin/gunicorn ssbc.wsgi:application -b 127.0.0.1:8000 --reload</br>
root      3693     1 12 21:15 ?        00:01:30 python simdht_worker.py</br>
root      3694     1  0 21:15 ?        00:00:00 python index_worker.py</br>
kill之后再kill下面几个进程</br>
ps -ef |grep search</br>
root      3467     1  0 21:03 ?        00:00:00 searchd --config ./sphinx.conf</br>
root      3468  3467  0 21:03 ?        00:00:02 searchd --config ./sphinx.conf</br>
接着修改配置文件</br>
vim /root/ssbc/sphinx.conf</br>
增加数据库的密码</br>
    sql_host                = 127.0.0.1</br>
    sql_user                = root</br>
    sql_pass                = </br>
    sql_db                  = ssbc</br>
    sql_port                = 3306  # optional, default is 3306</br>
vi /root/ssbc/workers/index_worker.py
</br>
SRC_HOST = '127.0.0.1'</br>
SRC_USER = 'root'</br>
SRC_PASS = ''</br>
DST_HOST = '127.0.0.1'</br>
DST_USER = 'root'</br>
DST_PASS = ''</br>
上面两个密码都要修改</br>
vi /root/ssbc/workers/simdht_worker.py</br>

DB_HOST = '127.0.0.1'</br>
DB_USER = 'root'</br>
DB_PORT = 3306</br>
DB_PASS = ''</br>
DB_NAME = 'ssbc'</br>
BLACK_FILE = 'black_list.txt'</br>

vim /root/ssbc/ssbc/settings.py</br>
修改下面，root后面加上数据库密码</br>
DATABASES = {</br>

'default': {</br>
    'ENGINE': 'django.db.backends.mysql',</br>
    'NAME': 'ssbc',</br>
    'HOST': '127.0.0.1',</br>
    'PORT': 3306,</br>
    'USER': 'root',</br>
    'PASSWORD': 'magrco.com',</br>
    'OPTIONS': {</br>
       "init_command": "SET storage_engine=MYISAM",</br>
    }</br>
}</br>
}</br>

<h2>关于数据迁移</h2></br>
这个其实好办先在新的机器上执行脚本，执行完成之后删除数据库建立新的ssbc数据库记住编码要utf-8的，之后把老的数据库导入新的就可以了</br>
<h2>其他的使用技巧</h2></br>

1.必须centos7吗？</br>
非常建议使用centos7，centos6可能会有意想不到的错误</br>
2.如何设置首页关键字？</br>
登录管理员后台，点击Rec keywordss，右上角新增</br>
3.怎么查看入库的文件？</br>
登录管理员后台，点击 Hashs </br>
4.怎么查看每天入库了多少文件，以便清楚入库效率？</br>
登录管理员后台，点击 Status reports </br>
5.如何确认web服务器、采集、入库正在运行？</br>
运行 ps -ef|grep python|grep -v grep</br>
结果里面有</br>
gunicorn ssbc.wsgi:application -b 127.0.0.1:8000 --reload </br>
python simdht_worker.py  </br>
python index_worker.py</br>
即表示正在运行。</br>
——————————————————————————————————————</br>
去除搜索页 右下角广告</br>
root@localhost ssbc-master# cd web/static/js</br>
root@localhost js# vi ssbc.js   找到如下3行，在前面添加//进行注释，保存</br>
//        document.write('<script src="http://v.6dvip.com/ge/?s=47688"><\/script>');</br>
//            document.writeln("<script language=\"JavaScript\" type=\"text/javascript\" src=\"http://js.6dad.com/js/xiaoxia.js\"></script>");</br>
//           document.writeln("<script language=\"JavaScript\" type=\"text/javascript\" src=\"http://js.ta80.com/js/12115.js\"></script>");</br>
—————————————————————————————————————</br>
如何修改扩展名归类？</br>
workers/metautils.py文件中有如下代码：</br>
def get_category(ext):</br>
ext = ext + '.'</br>
cats = {</br>
    u'video': '.avi.mp4.rmvb.m2ts.wmv.mkv.flv.qmv.rm.mov.vob.asf.3gp.mpg.mpeg.m4v.f4v.',</br>
    u'image': '.jpg.bmp.jpeg.png.gif.tiff.',</br>
    u'document': '.pdf.isz.chm.txt.epub.bc!.doc.ppt.',</br>
    u'music': '.mp3.ape.wav.dts.mdf.flac.',</br>
    u'package': '.zip.rar.7z.tar.gz.iso.dmg.pkg.',</br>
    u'software': '.exe.app.msi.apk.'</br>
}</br>
意思是：扩展名为.exe、.app、.msi、,.apk的文件都属于software类型。</br>
如果你把u'software': '.exe.app.msi.apk.' 改为 u'software': 'app.msi.apk.'，那么exe将会被归为other类型。</br>
所以在这里修改归类设置。</br>
——————————————————————————————————————</br>
如何禁止某些格式/分类的文件入库？</br>
workers/metadata.py文件中有如下代码：</br>
info'extension' = metautils.get_extension(bigfname).lower()</br>
info'category' = metautils.get_category(info'extension')</br>
所以如果你要排除扩展名为.exe的文件，或者类型为software，可以在上面代码后面加上</br>
##########这是增加的过滤-开始############</br>
#按扩张名过滤，禁止扩展名为.exe的入库</br>
if info'extension' == 'exe':</br>
return # 直接返回，跳过下面的入库</br>
#按文件类型过滤，禁止类型为software的入库</br>
if info'category' == 'software':</br>
return</br>
#禁止类型为other的入库</br>
if info'category' == 'other':</br>
return</br>
——————————————————————————————————————</br>
如何重建索引？</br>
第一步：</br>
删除/data目录</br>
第二步：</br>
进入数据库，把search_hash表中所有记录的tagged字段置为0。</br>
UPDATE search_hash SET tagged=0</br>
然后启动sphinx、index_worker.py。</br>
——————————————————————————————————————</br>
MySQL server has gone away提示怎么办？</br>
ssbc 运行一段时间后，大概半个小时，就莫名奇妙停止不爬了。 错误提示如下：</br>
MySQL server has gone away</br>
通过错误提示可以看出，其实是ssbc与mysql(maridb)断开连接了，导致程序异常，当然就插入不了数据了。</br>
有3种解决办法：</br>
方法1是写个脚本，定时重启爬虫。</br>
方法2是修改下代码，当mysql断开连接时，再次重连mysql就可以拉。</br>
方法3是修改Mysql配置，将闲置时间wait_timeout设置长一点。</br>
——————————————————————————————————————</br>
哪里设置爬虫线程？让爬虫爬快/慢点？</br>
在workers/simdht_worker.py里面把MAX_QUEUE_LT、MAX_QUEUE_PT、max_node_qsize设大/小一点。</br>
如何关闭调试模式？设置404页面？</br>
——————————————————————————————————————</br>
如何在搜索结果页面添加迅雷链接？</br>
在web/views.py文件加入以下代码生成迅雷链接:</br>
import base64</br>
xunleiurl = 'AAmagnet:?xt=urn:btih:' + d'info' + 'ZZ'</br>
d'xunlei_url' = 'thunder://' + base64.b64encode(xunleiurl)</br>
可以在模板中用“ {{xunlei_url}} ”调用。位置要放在return render(request, 'info.html', d)的前面。</br>
——————————————————————————————————————</br>
SSBC如何搬家？</br>
数据库用mysqldump导出sql，在新服务器上运行一键包，再导入刚才的sql。</br>
——————————————————————————————————————</br>
提示duplicate id 'xxxx'解决办法</br>
进入数据库，执行语句</br>
update search_hash set tagged=True where id=xxxx;</br>
——————————————————————————————————————</br>
搜索中文报错</br>
解决办法：</br>
如果是centos7系统，修改/usr/lib64/python2.7/site.py</br>
vi  /usr/lib64/python2.7/site.py</br>
在import sys下添加2行：</br>
reload(sys)</br>
sys.setdefaultencoding('utf8')</br>
——————————————————————————————————————</br>
如何关闭调试模式？设置404页面</br>
ssbc 是 基于 django 框架写的如何关闭调试模式呢？</br>
在 ssbc/setting.py 中将 DEBUG = True 改为 DEBUG = False</br>
这样就关闭了调试模式，但还没完。现在访问，会直接出现 500 的错误，这是静态资源造成的</br>
下面给出解决方案</br>
在 ssbc/setting.py 中将</br>
STATIC_ROOT = os.path.join(BASE_DIR, 'www/static')</br>
修改为</br>
STATIC_ROOT = os.path.join(BASE_DIR, 'web/static')</br>
将下面这行注释掉</br>
STATICFILES_STORAGE = 'django.contrib.staticfiles.storage.ManifestStaticFilesStorage'</br>
在 ssbc/urls.py 中导入 settings 包，然后再 urlpatterns 中添加</br>
url(r'^static/(?P<path>.*)$','django.views.static.serve',{'document_root':settings.STATIC_ROOT})</br>
现在访问出现404后，显示的是我们制定的404页面，而不是错误页面了。</br>

注意：在修改正在运行的程序是一定要结束运行再修改。</br>
更多信息请访问：http://wiz.im  http://magrco.com</br>
