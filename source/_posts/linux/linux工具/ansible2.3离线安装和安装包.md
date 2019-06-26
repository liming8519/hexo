---
title: ansible2.3离线安装和离线安装包
tags:
  - tool
categories:
  - linux
date: 2019-06-26 05:00:00
---
> ansible2.3离线安装和离线安装包
<!-- more -->
```
## 初始化

yum install python-devel openssl-devel

## 1.创建文件夹
#mkdir -p /usr/local/src/ansible

## 2. 安装setuptools
cd /usr/local/src/ansible
tar xvzf setuptools-7.0.tar.gz
cd setuptools-7.0
python setup.py install

## 3. 安装pycrypto
cd /usr/local/src/ansible
tar xvzf pycrypto-2.6.1.tar.gz
cd pycrypto-2.6.1
python setup.py install

## 4. 安装yaml
cd /usr/local/src/ansible
tar xvzf yaml-0.1.5.tar.gz
cd yaml-0.1.5
./configure --prefix=/usr/local
make --jobs=`grep processor /proc/cpuinfo | wc -l`
make install

## 5. 安装PyYAML
cd /usr/local/src/ansible
tar xvzf PyYAML-3.11.tar.gz
cd PyYAML-3.11  
python setup.py install  
  
## 6. 安装MarkupSafe  
cd /usr/local/src/ansible  
tar xvzf MarkupSafe-0.9.3.tar.gz
cd MarkupSafe-0.9.3  
python setup.py install

## 7. 安装Jinja2  
cd /usr/local/src/ansible  
tar xvzf Jinja2-2.7.3.tar.gz  
cd Jinja2-2.7.3  
python setup.py install  
  
## 8. 安装ecdsa  
cd /usr/local/src/ansible  
tar xvzf ecdsa-0.11.tar.gz
cd ecdsa-0.11  
python setup.py install  
cd /usr/local/src/ansible
tar xvzf paramiko-1.15.1.tar.gz
cd paramiko-1.15.1
python setup.py install

## 9. 安装simplejson
cd /usr/local/src/ansible
tar xvzf simplejson-3.6.5.tar.gz
cd simplejson-3.6.5
python setup.py install

## 10. 安装ansible
cd /usr/local/src/ansible
tar xvzf ansible-2.3.1.0.tar.gz
cd ansible-2.3.1.0
python setup.py install


## 11. 安装sshpass
cd /usr/local/src/ansible
cd sshpass-1.06
./configure
make
make install
```
## 安装包位置

[ansible离线安装包](https://raw.githubusercontent.com/zixujing/book1.github.io/master/code/ansible%E7%A6%BB%E7%BA%BF%E5%AE%89%E8%A3%85%E5%8C%85.zip)
## 配置例子
### 统一成一个命令
```
#! /bin/perl
%command=(

#所有机器就绪状态

1,qq \ansible all -m ping\,

#服务监控

2, qq \ansible http-service -m shell -a "systemctl status tomcat8_pf | grep Active:"\,
3, qq \ansible http-service -m shell -a "systemctl status tomcat8_sdk | grep Active:"\,
4, qq \ansible http-service -m shell -a "df -H /dev/mapper/cl-home;du -sm /home/hch54"\,

#服务启动与结束
5, qq \ansible 72 -m shell -a "systemctl start tomcat8_pf"\,
6, qq \ansible 73 -m shell -a "systemctl start tomcat8_pf"\,
7, qq \ansible 74 -m shell -a "systemctl start tomcat8_pf"\,
8, qq \ansible 72 -m shell -a "systemctl stop tomcat8_pf"\,
9, qq \ansible 73 -m shell -a "systemctl stop tomcat8_pf"\,
10, qq \ansible 74 -m shell -a "systemctl stop tomcat8_pf"\,

11, qq \ansible 72 -m shell -a "systemctl start tomcat8_sdk"\,
12, qq \ansible 73 -m shell -a "systemctl start tomcat8_sdk"\,
13, qq \ansible 74 -m shell -a "systemctl start tomcat8_sdk"\,
14, qq \ansible 72 -m shell -a "systemctl stop tomcat8_sdk"\,
15, qq \ansible 73 -m shell -a "systemctl stop tomcat8_sdk"\,
16, qq \ansible 74 -m shell -a "systemctl stop tomcat8_sdk"\,


#nginx监控
17,qq \ansible nginx -m shell -a "systemctl status nginx | grep Active:"\,
18,qq \ansible nginx -m shell -a "systemctl status keepalived | grep Active:"\,



19,qq \ansible 74 -m shell -a "systemctl start nginx"\,
20,qq \ansible 74 -m shell -a "systemctl stop nginx"\,
21,qq \ansible 48 -m shell -a "systemctl start nginx"\,
22,qq \ansible 48 -m shell -a "systemctl stop nginx"\,



23,qq \ansible 48 -m shell -a "nginx -s reload"\,

#kafka

24,qq /ansible kafka -m shell -a "ss -nlt | egrep \\"(:2181)|(:9092)\\""/,
25,qq \ansible 127 -m shell -a "source /root/.bash_profile;/opt/kafka_2.12-2.1.1/bin/kafka-server-start.sh -daemon /opt/kafka_2.12-2.1.1/config/server_sasl.properties"\,
26,qq \ansible 128 -m shell -a "source /root/.bash_profile;/opt/kafka_2.12-2.1.1/bin/kafka-server-start.sh -daemon /opt/kafka_2.12-2.1.1/config/server_sasl.properties"\,
27,qq \ansible 134 -m shell -a "source /root/.bash_profile;/opt/kafka_2.12-2.1.1/bin/kafka-server-start.sh -daemon /opt/kafka_2.12-2.1.1/config/server_sasl.properties"\,


28,qq \ansible 127 -m shell -a "source /root/.bash_profile;/opt/kafka_2.12-2.1.1/bin/zookeeper-server-start.sh -daemon /opt/kafka_2.12-2.1.1/config/zookeeper.properties"\,
29,qq \ansible 128 -m shell -a "source /root/.bash_profile;/opt/kafka_2.12-2.1.1/bin/zookeeper-server-start.sh -daemon /opt/kafka_2.12-2.1.1/config/zookeeper.properties"\,
30,qq \ansible 134 -m shell -a "source /root/.bash_profile;/opt/kafka_2.12-2.1.1/bin/zookeeper-server-start.sh -daemon /opt/kafka_2.12-2.1.1/config/zookeeper.properties"\,

#mysql
31,q \ansible 26 -m shell -a "source /root/.bash_profile;mysql -uroot -p12qWe@#$% -e'show master status'"\,
32,qq !ansible 25 -m shell -a "source /root/.bash_profile;mysql -uroot -p123456 -e'show slave status\G'"!,


33,qq \ansible 26 -m shell -a "systemctl status mysqld"\,
34,qq \ansible 25 -m shell -a "systemctl status mysql"\,

35,qq \ansible 26 -m shell -a "systemctl start mysqld"\,
36,qq \ansible 25 -m shell -a "systemctl start mysql"\,
37,qq \ansible 26 -m shell -a "systemctl stop mysqld"\,
38,qq \ansible 25 -m shell -a "systemctl stop mysql"\,

39,qq !ansible 25 -m shell -a "source /root/.bash_profile;mysql -uroot -p123456 -e'start slave'"!,

#oracle
40,qq \ansible 25 -m shell -a "source /etc/profile; lsnrctl status|grep READY"\,
41,qq \ansible 25 -m shell -a "source /etc/profile; lsnrctl stop"\,
42,qq \ansible 25 -m shell -a "source /etc/profile; lsnrctl start"\,

43,q !ansible 25 -m shell --become-user=oracle -b -a "source /etc/profile; echo -e  'select open_mode,dbid from v\\$database;' | sqlplus / as sysdba | grep 1526735889"!,

44,qq \ansible 25 -m shell --become-user=oracle -b -a "source /etc/profile;dbstart"\,
45,qq \ansible 25 -m shell --become-user=oracle -b -a "source /etc/profile;dbshut"\,

#redis
46,qq _ansible redis -m shell -a "ss -nlt | egrep \\"(:6500)|(:6600)\\""_,
47,qq \ansible 81 -m shell -a "redis-server /home/redis_copy/6500/redis.conf"\,
48,qq \ansible 81 -m shell -a "redis-sentinel /home/redis_copy/6600/sentinel.conf"\,
49,qq \ansible 84 -m shell -a "redis-server /home/redis_copy/6500/redis.conf"\,

#activemq
50,qq \ansible 85 -m shell -a "systemctl status activemq|grep Active"\,
51,qq \ansible 85 -m shell -a "systemctl stop activemq"\,
52,qq \ansible 85 -m shell -a "systemctl start activemq"\,
53,qq \ansible 85 -m shell -a "df -h /dev/mapper/cl-home"\,

#文件服务器
54,qq \ansible 83 -m shell -a "df -h | grep -v tmp"\,
55,qq \ansible 83 -m shell -a "systemctl status fdfs_storaged |grep Active ; systemctl status fdfs_trackerd |grep Active"\,

56,qq \ansible 83 -m shell -a "systemctl stop fdfs_storaged ; systemctl stop fdfs_trackerd "\,
57,qq \ansible 83 -m shell -a "systemctl start fdfs_storaged ; systemctl start fdfs_trackerd "\,

#mpp
58,qq \ansible 135 -m shell --become-user=gpadmin -b -a "source /home/gpadmin/conf/gpsh;gpstate"\,
59,qq \ansible 135 -m shell --become-user=gpadmin -b -a "source /home/gpadmin/conf/gpsh;gpstart"\,
60,qq \ansible 135 -m shell --become-user=gpadmin -b -a "source /home/gpadmin/conf/gpsh;gpstop"\



#结束括号
);
#for debug
#print $command{$ARGV[0]},"\n";
$info=qx | $command{$ARGV[0]}|;
print $info;


```
### 做成一个网页
```
#! /usr/bin/python
#coding=utf8
from BaseHTTPServer import HTTPServer,BaseHTTPRequestHandler
import urllib
import commands
class ServerHttp(BaseHTTPRequestHandler):
	def do_GET(self):
		path=self.path
		#print path
		query=urllib.splitquery(path)
		#print query[1]
		#print query[0]
		body_data=None
		if(query[0]=="/system" and query[1] is not None):
			body_data=commands.getstatusoutput("hch_state %s"%query[1])
			#print "hch_state %s"%query[1]
		self.send_response(200)
		self.send_header("Content-type","text/html")
		self.send_header("(^_^)","?? !")
		self.end_headers()
		buf1='''
		<html>
		<head>
		<meta http-equiv="Content-type" content="text/html;charset=UTF-8">
		<link rel="icon" href="data:;base64,="/>
		<title>(^_^)</title>
		</head>
		<body>
		<h1>请点击以下各个链接</h1>
		<font size="20">
		<a href="?tag=1">1</a>
		<a href="?tag=2">2</a>
		<a href="?tag=3">3</a>
		<a href="?tag=4">4</a>
		<a href="?tag=5">5</a>
		<a href="?tag=6">6</a>
		<a href="?tag=7">7</a>
		<a href="?tag=8">8</a>
		<a href="?tag=9">9</a>
		<a href="?tag=10">10</a>
		<a href="?tag=11">11</a>
		<a href="?tag=12">12</a>
		<a href="?tag=13">13</a>
		<a href="?tag=14">14</a>
		<a href="?tag=15">15</a>
		<a href="?tag=16">16</a>
		<a href="?tag=17">17</a>
		<a href="?tag=18">18</a>
		<a href="?tag=19">19</a>
		<a href="?tag=20">20</a></br>
		<a href="?tag=21">21</a>
		<a href="?tag=22">22</a>
		<a href="?tag=23">23</a>
		<a href="?tag=24">24</a>
		<a href="?tag=25">25</a>
		<a href="?tag=26">26</a>
		<a href="?tag=27">27</a>
		<a href="?tag=28">28</a>
		<a href="?tag=29">29</a>
		<a href="?tag=30">30</a>
		<a href="?tag=31">31</a>
		<a href="?tag=32">32</a>
		<a href="?tag=33">33</a>
		<a href="?tag=34">34</a>
		<a href="?tag=35">35</a>
		<a href="?tag=36">36</a>
		<a href="?tag=37">37</a>
		<a href="?tag=38">38</a>
		<a href="?tag=39">39</a>
		<a href="?tag=40">40</a></br>
		<a href="?tag=41">41</a>
		<a href="?tag=42">42</a>
		<a href="?tag=43">43</a>
		<a href="?tag=44">44</a>
		<a href="?tag=45">45</a>
		<a href="?tag=46">46</a>
		<a href="?tag=47">47</a>
		<a href="?tag=48">48</a>
		<a href="?tag=49">49</a>
		<a href="?tag=50">50</a>
		<a href="?tag=51">51</a>
		<a href="?tag=52">52</a>
		<a href="?tag=53">53</a>
		<a href="?tag=54">54</a>
		<a href="?tag=55">55</a>
		<a href="?tag=56">56</a>
		<a href="?tag=57">57</a>
		<a href="?tag=58">58</a>
		<a href="?tag=59">59</a>
		<a href="?tag=60">60</a></br>
		</font>
<font size=8>
<table border="1">
		
<tr>
<td>1.所有服务器是否在线</td>
<td>2.核查平台服务状态</td>
<td>3.sdk服务状态</td>
<td>4.平台服务磁盘使用</td>
<td>5.核查平台服务启动72</td>
<td>6.核查平台服务启动73</td>
<td>7.核查平台服务启动74</td>
<td>8.核查平台服务关闭72</td>
<td>9.核查平台服务关闭73</td>
<td>10.核查平台服务关闭74</td>
</tr>

<tr>
<td>11.sdk服务启动72</td>
<td>12.sdk服务启动73</td>
<td>13.sdk服务启动74</td>
<td>14.sdk服务关闭72</td>
<td>15.sdk服务关闭73</td>
<td>16.sdk服务关闭74</td>
<td>17.nginx服务状态</td>
<td>18.keepalived服务状态</td>
<td>19.启动nginx74</td>
<td>20.关闭nginx74</td>
</tr>

<tr>
<td>21.启动nginx48</td>
<td>22.关闭nginx48</td>
<td>23.重载nginx配置48</td>
<td>24.kafka状态</td>
<td>25.启动kafka127</td>
<td>26.启动kafka128</td>
<td>27.启动kafka134</td>
<td>28.启动zookeeper127</td>
<td>29.启动zookeeper128</td>
<td>30.启动zookeeper1134</td>
</tr>
<tr>
<td>31.26mysql master status</td>
<td>32.25mysql slave status</td>
<td>33.26mysql服务状态</td>
<td>34.25mysql服务状态</td>
<td>35.26mysql启动</td>
<td>36.25mysql启动</td>
<td>37.26mysql关闭</td>
<td>38.25mysql关闭</td>
<td>39.25mysql start slave</td>
<td>40.oracle监听状态</td>
</tr>
<tr>
<td>41.oracle监听结束</td>
<td>42.oracle监听启动</td>
<td>43.oracle状态</td>
<td>44.启动oracle</td>
<td>45.关闭oracle</td>
<td>46.redis状态</td>
<td>47.81启动redis</td>
<td>48.81启动哨兵</td>
<td>49.84启动redis</td>
<td>50.activemq状态</td>
</tr>
<tr>
<td>51.结束activemq</td>
<td>52.启动activemq</td>
<td>53.activemq磁盘使用</td>
<td>54.83文件服务磁盘使用</td>
<td>55.fastdfs状态</td>
<td>56.关闭fastdfs</td>
<td>57.启动fastdfs</td>
<td>58.mpp状态</td>
<td>59.mpp启动</td>
<td>60.mpp关闭</td>
</tr>
</table>
</font>
		<hr>
		<h1>下面是返回信息</h1>
		<p align="center">
		<textarea readonly rows="100" cols="180">'''
		buf2='''</textarea>
		</p>
		</body>
		</html>
		'''
		#print body_data
		body_data=body_data if(body_data is not None) else (0,"")
		#print body_data
	
		self.wfile.write(buf1+body_data[1]+buf2)
def start_server(port):
	http_server=HTTPServer(('',int(port)),ServerHttp)
	http_server.serve_forever()
if __name__=="__main__":
	start_server(12345)

```
