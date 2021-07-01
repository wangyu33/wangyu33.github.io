# QQ音乐关键字搜索并生成下载url

## 1. 获取搜索的歌单信息

​	通过在QQ音乐进行关键字搜索，并通过F12检测，发现网页给网站https://c.y.qq.com/soso/fcgi-bin/client_search_cp? 发送了一条get请求，便可以得到其关键词搜索的json信息，通过简化可以得到以下关键词搜索的网址：https://c.y.qq.com/soso/fcgi-bin/client_search_cp?new_json=1&remoteplace=txt.yqq.song&t=0&aggr=1&cr=1&w={}&format=json&platform=yqq.json

![QQ音乐1](C:\Users\wangyu\Desktop\博客\QQ音乐1.png)

​	通过检查其preview我们可以发现，其中包含了我们所需要搜索歌曲的信息，如下所示：

![QQ音乐2](C:\Users\wangyu\Desktop\博客\QQ音乐2.png)

​	通过修改w所对应的值便可以得到我们所需要的json数据。

## 2. 获取下载外链

### 2.1 观察网页数据传输

​	类似于获取歌单信息，打开一首歌并播放，按传输数据的大小排序，我们可以发现这样一条大小为3M多的get请求，毫无疑问，这便是我们所收听的歌曲。下载的url由guid、vkey、uin、fromtag组成。

![QQ音乐3](C:\Users\wangyu\Desktop\博客\QQ音乐3.png)

​	将Network切换到XHR，按经过一番查看，会看到这个文件，这样我们便可以直接得到下载所需的guid、vkey、uin、fromtag参数。

![QQ音乐4](C:\Users\wangyu\Desktop\博客\QQ音乐4.png)

​	通过检查其headers，可以获得其发送的get请求网址：

![QQ音乐5](C:\Users\wangyu\Desktop\博客\QQ音乐5.png)

​	为了方便观察，我们对其进行解码：

![QQ音乐6](C:\Users\wangyu\Desktop\博客\QQ音乐6.png)

​	由于uin涉及测试的用户信息，在这对其进行打码处理，最后解码出来的url经过可以表示为'https://u.y.qq.com/cgi-bin/musicu.fcg?data={"req_0":{"module":"vkey.GetVkeyServer","method":"CgiGetVkey","param":{"guid":"4095854469","songmid":["%s"],"songtype":[0],"uin":"0","loginflag":1,"platform":"20"}},"comm":{"uin":0,"format":"json","ct":24,"cv":0}}'

​	经过检查guid的数值似乎是固定值，因此这些数值均可以直接使用，仅需改变songmid的数值即可。而songmid的数值可以通过1中的json数据直接获取。因此，将'http://dl.stream.qqmusic.qq.com/'或者'http://ws.stream.qqmusic.qq.com/'直接与purl进行拼接即可。

### 2.2 VIP音乐处理

​	在网页进行点击播放VIP音乐时会显示无法进行播放，针对这种情况，我们需要使用chrome浏览器模拟客户端登录。

![QQ音乐7](C:\Users\wangyu\Desktop\博客\QQ音乐7.png)

![QQ音乐8](C:\Users\wangyu\Desktop\博客\QQ音乐8.png)

​	点击小手机图案便可以模拟手机登录，观察网页源码，惊喜的发现下载url就这样明晃晃的显示在眼前，像是大喜之夜的新娘，然而，当我使用request去get网页源码，发现非会员的url是可以直接显示的，然而会员的url我怎样也无法get到，由于笔者的js水平不行，不太懂这里面的玄机，待以后有机会学成归来，一探其中玄奥。虽然requests不行，但我们还是可以使用selenium来暴力获取，通过正则处理可以直接提取我们所需要的歌曲下载url。

​	经我测试，获取完整的vip歌曲需要会员的cookie，否则尽可以获取片段。

## 3. 完整代码

```python
import os,json
import re
from selenium import webdriver
from urllib.parse import quote
import time
import requests
import pickle
import random
def getcookie():
    driver = webdriver.Chrome('chromedriver.exe')
    driver.get('https://y.qq.com/')
    time.sleep(30)
    cookie = driver.get_cookies()
    driver.quit()
    with open('QQyy_cookie.pkl', 'wb') as f:
        pickle.dump(cookie, f)
    return cookie

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

class QQ_Music():
    def __init__(self):
        self.get_music_url='https://c.y.qq.com/soso/fcgi-bin/client_search_cp?new_json=1&remoteplace=txt.yqq.song&t=0&aggr=1&cr=1&w={}&format=json&platform=yqq.json'
        self.get_song_url='https://u.y.qq.com/cgi-bin/musicu.fcg?data={"req_0":{"module":"vkey.GetVkeyServer","method":"CgiGetVkey","param":{"guid":"4095854469","songmid":["%s"],"songtype":[0],"uin":"0","loginflag":1,"platform":"20"}},"comm":{"uin":0,"format":"json","ct":24,"cv":0}}'
        self.download_url='http://dl.stream.qqmusic.qq.com/'
        self.headers = {'User-Agent': set_user_agent()}

    def parse_url(self,url):
        response = requests.get(url)
        return response.content.decode()

    def get_music_list(self,keyword):
        music_dirt=json.loads(self.parse_url(self.get_music_url.format(quote(keyword))))
        music_list=music_dirt['data']['song']['list']

        return music_list

    def get_download_url(self, name):
        music_list = self.get_music_list(name)
        words = 'QQ音乐：\n'
        if not os.path.exists("QQyy_cookie.pkl"):
            getcookie()
        cookie = pickle.load(open("QQyy_cookie.pkl", "rb"))
        cookies = {}
        for c in cookie:
            cookies[c['name']] = c['value']
        cnt = 0
        for music in music_list:
            #歌手
            if cnt == 3:
                break
            cnt = cnt + 1
            music_id = music['mid']
            music_name = music['name']
            music_author = music['singer'][0]['name']
            music_album = music['album']['name']
            words = words + '歌名:' + music_name + '\n'
            words = words + '歌手:' + music_author + '\n'
            words = words + '专辑名:' + music_album + '\n'
            song_url = self.get_song_url % music_id
            response = requests.get(song_url, headers=self.headers, cookies=cookies).content
            song_dirt = json.loads(response)
            download_url = self.download_url + song_dirt["req_0"]["data"]["midurlinfo"][0]["purl"]
            if download_url != self.download_url:
                words = words + '下载链接:' + download_url + '\n'
            else:
                url = 'https://i.y.qq.com/v8/playsong.html?ADTAG=newyqq.song&songmid={}'.format(music_id)
                options = webdriver.ChromeOptions()
                options.add_argument('--no-sandbox')  # 解决DevToolsActivePort文件不存在的报错
                options.add_argument('window-size=1600x900')  # 指定浏览器分辨率
                options.add_argument('--disable-gpu')  # 谷歌文档提到需要加上这个属性来规避bug
                options.add_argument('--hide-scrollbars')  # 隐藏滚动条, 应对一些特殊页面
                options.add_argument('blink-settings=imagesEnabled=false')  # 不加载图片, 提升速度
                options.add_argument('--headless')  # 浏览器不提供可视化页面. linux下如果系统不支持可视化不加这条会启动失败
                mobileEmulation = {'deviceName': 'Galaxy S5'}
                options.add_experimental_option('mobileEmulation', mobileEmulation)
                driver = webdriver.Chrome(options=options, executable_path='./chromedriver')
                driver.get(url)
                cookie = pickle.load(open("QQyy_cookie.pkl", "rb"))
                driver.delete_all_cookies()
                for cookie_ in cookie:
                    driver.add_cookie(cookie_)
                time.sleep(3)
                driver.refresh()
                response = driver.page_source
                driver.quit()
                pattern = re.compile('<audio id="h5audio_media" height="0" width="0" src="(.*?)"',
                                     re.S)
                m_url = pattern.findall(response)
                if m_url:
                    m_url[0] = m_url[0].replace(';','&')
                    words = words + '下载链接:' + m_url[0] + '\n'
                else:
                    words = words + '下载链接:' + '无' + '\n'
        return words

if __name__ == '__main__':
    qqmusic=QQ_Music()
    print(qqmusic.get_download_url('许嵩浅唱'))
```

