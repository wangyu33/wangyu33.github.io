# 酷狗音乐关键字搜索并生成下载url

## 1. 获取搜索的歌单信息

​	通过在酷狗音乐进行关键字搜索，并通过F12检测，发现网页给网站发送了get请求。

```
https://complexsearch.kugou.com/v2/search/song?callback=callback123&keyword=%E7%A8%BB%E9%A6%99&page=1&pagesize=30&bitrate=0&isfuzzy=0&tag=em&inputtype=0&platform=WebFilter&userid=-1&clientver=2000&iscorrection=1&privilege_filter=0&srcappid=2919&clienttime=1598343363775&mid=1598343363775&uuid=1598343363775&dfid=-&signature=09553BE20E21CF594C911530E4F71A07 
```

​	其中keyword是搜索关键字，clienttime，mid，uuid均是时间戳，其他数值除signature经过检查均为固定值，针对signature的解码过程如下：

![image-20200825173149864](C:\Users\wangyu\AppData\Roaming\Typora\typora-user-images\image-20200825173149864.png)

通过设置断点，可以获得signature，其获取流程如下：

```
str = {"NVPh5oo715z5DIWAeQlhMDsWXXQV4hwt"
"bitrate=0"
"callback=callback123"
"clienttime=1598347379352"
"clientver=2000"
"dfid=-"
"inputtype=0"
"iscorrection=1"
"isfuzzy=0"
"keyword=稻香"
"mid=1598347379352"
"page=1"
"pagesize=30"
"platform=WebFilter"
"privilege_filter=0"
"srcappid=2919"
"tag=em"
"userid=-1"
"uuid=1598347379352"
"NVPh5oo715z5DIWAeQlhMDsWXXQV4hwt"}
md5 = hashlib.md5()
md5.update(''.join(str).encode())
signature = md5.hexdigest()
# keyword是搜索关键字，clienttime，mid，uuid均是时间戳
```



时间戳为毫秒级（13位），需要自己实现函数，于是本人为了省事直接使用了前人用过的搜索url，似乎还可以使用(这样可以增加处理速度)：

```
https://songsearch.kugou.com/song_search_v2?keyword={}&platform=WebFilter
```



![酷狗1](C:\Users\wangyu\Desktop\博客\酷狗1.png)

​	

## 2. 获取下载外链

​	类似于获取歌单信息，打开一首歌并播放，按传输数据的大小排序，可是我们并没有发现存在较大的数据量的请求，经过慢慢检查，筛选出这条信息内包含我们所需的播放url：

![酷狗2](C:\Users\wangyu\Desktop\博客\酷狗2.png)

​	通过对header中的url进行简化处理，发现获取音乐下载url的url形式如下所示：

```
https://wwwapi.kugou.com/yy/index.php?r=play/getdata&hash={}&mid={}
```

​	hash可以通过1中的json数据获得，mid可以通过在'abcdefghijklmnopqrstuvwxyz0123456789'中任选4个字符，并使用md5加密获得。

​	由于没有酷狗VIP的账号，因此无法仅能下载VIP音乐片段。

## 3. 完整代码

```python
 import urllib.request,os,json
from urllib.parse import quote
import random
import requests
import time
from selenium import webdriver
import hashlib

def set_user_agent():
    USER_AGENTS = [
        "Mozilla/5.0 (compatible; MSIE 9.0; Windows NT 6.1; Win64; x64; Trident/5.0; .NET CLR 3.5.30729; .NET CLR 3.0.30729; .NET CLR 2.0.50727; Media Center PC 6.0)",
        "Mozilla/5.0 (compatible; MSIE 8.0; Windows NT 6.0; Trident/4.0; WOW64; Trident/4.0; SLCC2; .NET CLR 2.0.50727; .NET CLR 3.5.30729; .NET CLR 3.0.30729; .NET CLR 1.0.3705; .NET CLR 1.1.4322)",
        "Mozilla/4.0 (compatible; MSIE 7.0b; Windows NT 5.2; .NET CLR 1.1.4322; .NET CLR 2.0.50727; InfoPath.2; .NET CLR 3.0.04506.30)",
        "Mozilla/5.0 (Windows; U; Windows NT 5.1; zh-CN) AppleWebKit/523.15 (KHTML, like Gecko, Safari/419.3) Arora/0.3 (Change: 287 c9dfb30)",
        "Mozilla/5.0 (X11; U; Linux; en-US) AppleWebKit/527+ (KHTML, like Gecko, Safari/419.3) Arora/0.6",
        "Mozilla/5.0 (Windows; U; Windows NT 5.1; en-US; rv:1.8.1.2pre) Gecko/20070215 K-Ninja/2.1.1",
        "Mozilla/5.0 (Windows; U; Windows NT 5.1; zh-CN; rv:1.9) Gecko/20080705 Firefox/3.0 Kapiko/3.0",
        "Mozilla/5.0 (X11; Linux i686; U;) Gecko/20070322 Kazehakase/0.4.5"
    ]
    user_agent = random.choice(USER_AGENTS)
    return user_agent

class KuGou():
    def __init__(self):
        self.get_music_url='https://songsearch.kugou.com/song_search_v2?keyword={}&platform=WebFilter'
        self.get_song_url='https://wwwapi.kugou.com/yy/index.php?r=play/getdata&hash={}'
        self.headers = {'User-Agent': set_user_agent()}

    def parse_url(self,url):
        response = requests.get(url,headers = self.headers)
        return response.content.decode()

    def get_music_list(self,keyword):
        url = self.get_music_url.format(quote(keyword))
        music_dirt = json.loads(self.parse_url(url))
        music_list = music_dirt['data']['lists']
        return music_list

    def creat_mid(self):
        md5 = hashlib.md5()
        # 随机生成4位随机的字符列表 范围为a-z 0-9
        n = random.sample('abcdefghijklmnopqrstuvwxyz0123456789', 4)
        # 将列表元素拼接为字符串
        n = ''.join(n)
        # 将字符串编码后更新到md5对象里面
        md5.update(n.encode())
        # 调用hexdigest获取加密后的返回值
        result = md5.hexdigest()
        return result

    def get_download_url(self, key):
        music_list = self.get_music_list(key)
        cnt = 0
        words = '酷狗：\n'
        for music in music_list:
            if cnt == 3:
                break
            cnt = cnt + 1
            music_id = music['FileHash']
            # print(type(music_id))
            music_name = music['SongName']
            music_author = music['SingerName']
            music_album = music['AlbumName']
            words = words + '专辑名:' + music_album + '\n'
            words = words + '歌名:' + music_name + '\n'
            words = words + '歌手:' + music_author + '\n'
            song_url = self.get_song_url.format(music_id) + '&mid=' + self.creat_mid()
            response = json.loads(requests.get(song_url, headers = self.headers).content.decode())

            download_url = response['data']['play_url']
            if download_url:
                words = words + '下载链接:' + download_url + '\n'
            else:
                words = words + '下载链接:' + '无' + '\n'
        return words

if __name__ == '__main__':
    kugou=KuGou()
    print(kugou.get_download_url('许嵩浅唱'))
```

