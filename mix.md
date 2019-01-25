# BeautifulSoup4 print() 输出中文乱码解决方法

	import  requests
	from bs4 import BeautifulSoup  #pip install beautifulsoup4
 
BeautifulSoup 输出中文 => print cmd 默认编码是 Codepage 936
https://www.baidu.com/ 网页编码是 uft-8
导致 print() 输出乱码
解决方法:
让 r.encoding = 'gbk2312'; 编码为 gbk 此时输出正常.
 
你可以找出 Requests 使用了什么编码，并且能够使用 r.encoding 属性来改变它

	r= requests.get('https://www.baidu.com/');
	r.encoding = 'gbk2312';
	soup=BeautifulSoup(r.text,"html.parser") 
	print(soup.title);
 
写入到文件.

	fo = open('ttt.txt', "w")        
	fo.write(('\r' +soup.title.string + '\r\n'))  
	fo.close()       


# python3安装pip的方法之一
1. 点击[链接](https://bootstrap.pypa.io/get-pip.py),并下载`get-pip.py`文件;
2. 文件下载完成之后，cd到当前目录，并进行安装，`python3 get-pip.py`命令（如有必要使用超级权限）

