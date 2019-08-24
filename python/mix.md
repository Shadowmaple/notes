# BeautifulSoup4 print() 输出中文乱码解决方法

```python
import  requests
from bs4 import BeautifulSoup  #pip install beautifulsoup4
```

BeautifulSoup 输出中文 => print cmd 默认编码是 Codepage 936
https://www.baidu.com/ 网页编码是 uft-8
导致 print() 输出乱码
解决方法:
让 r.encoding = 'gbk2312'; 编码为 gbk 此时输出正常.

你可以找出 Requests 使用了什么编码，并且能够使用 r.encoding 属性来改变它

```python
r= requests.get('https://www.baidu.com/');
r.encoding = 'gbk2312';
soup=BeautifulSoup(r.text,"html.parser") 
print(soup.title);
```

写入到文件.

```python
fo = open('ttt.txt', "w")        
fo.write(('\r' +soup.title.string + '\r\n'))  
fo.close()       
```


