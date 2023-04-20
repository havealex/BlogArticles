@[TOC](Python爬取豆瓣电影 Top 250的图片)
# 一、分析豆瓣电影Top 250页面、分页的构成
![首页页面](https://img-blog.csdnimg.cn/ff01ed69ff494a459ef29519fd8cd4c6.png)


豆瓣电影页面:[https://movie.douban.com/top250](https://movie.douban.com/top250)
一共有10个分页，每个分页页面对应的链接地址为：
- https://movie.douban.com/top250
- https://movie.douban.com/top250?start=25&filter=
- https://movie.douban.com/top250?start=50&filter=
- https://movie.douban.com/top250?start=75&filter=
- https://movie.douban.com/top250?start=100&filter=
- https://movie.douban.com/top250?start=125&filter=
- https://movie.douban.com/top250?start=150&filter=
- https://movie.douban.com/top250?start=175&filter=
- https://movie.douban.com/top250?start=200&filter=
- https://movie.douban.com/top250?start=225&filter=

再来看一下每个图片卡片的具体组成，鼠标选择图片，在弹出的右键菜单中点击【检查】，会跳转到html源代码中的<a></a>标签：
```html
<a href="https://movie.douban.com/subject/1292052/">
<img width="100" alt="肖申克的救赎" src="https://img2.doubanio.com/view/photo/s_ratio_poster/public/p480747492.webp" class="">
</a>
```

# 二、安装Python3依赖库
- requests
- BeautifulSoup4

```shell
pip install requests BeautifulSoup4
```

# Python爬虫代码
## 版本1、使用Python爬取单个页面的所有电影图片
对应的Python代码`DoubanPictureDownloadVer01.py`如下：
```python
import requests
from bs4 import BeautifulSoup
headers ={
	'user-agent' : 'Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/111.0.0.0 Safari/537.36'
}
# 第一个分页地址
url = 'https://movie.douban.com/top250'
# 获取网页内容
html = requests.get(url=url, headers=headers).text
bs = BeautifulSoup(html, 'html.parser')
# 在获取的html内容中找到宽度为100的img元素列表
list_img = bs.find_all('img', attrs={'width':'100'})
# 遍历img图片列表
for i in list_img:
    # 获取电影图片下载地址
    img_url = i.get('src')
    print(img_url)
    # 获取电影名称
    movieName = i.get('alt')
    # 打印电影名称，电影图片下载地址
    print(movieName, img_url)
    # 将当前电源图片下载到当前目录下的images文件夹中
    with open('./images/' + movieName + '.jpg', 'wb') as f:
        f.write(requests.get(img_url).content)

```

## 版本2、获取豆瓣Top250所有页面的电影图片
对应的Python代码`doubanDownloadAllPictures.py`如下：
```python
import requests
from bs4 import BeautifulSoup
headers ={
	'user-agent' : 'Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/111.0.0.0 Safari/537.36'
}
# 电影总数
movieCnt = 0
# 由于每页有25个电影，一共有10个分页，所以电影的个数从0开始，截至到250，每个分页步长为25
for n in range(0, 250, 25):
	# url = 'https://movie.douban.com/top250'
    # 根据规律获取每个分页对应的url地址，其中后缀start=每页开始电影的索引 0,25,50,75...200,225
	url = 'https://movie.douban.com/top250?start='+str(n)
    print(url)
	html = requests.get(url=url, headers=headers).text
	bs = BeautifulSoup(html, 'html.parser')
	list_img = bs.find_all('img', attrs={'width':'100'})
    # 遍历img图片列表
	for i in list_img:
		# 每遍历一个电影，电影个数加1
		movieCnt = movieCnt + 1
        # 打印电影当前个数
		print(movieCnt)
		print("movieCnt: {:d}".format(movieCnt))
        # 获取电影图片下载地址
		img_url = i.get('src')
		print(img_url)
        # 获取电影名称
		movieName = i.get('alt')
        # 打印电影名称，电影图片下载地址
		print(movieName, img_url)
        # 将当前电源图片下载到当前目录下的images文件夹中
		with open('./images/' + movieName + '.jpg', 'wb') as f:
			f.write(requests.get(img_url).content)
```


