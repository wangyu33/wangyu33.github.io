# 网易云音乐关键字搜索并生成下载url

## 一、获取搜索的歌单信息

![网易云搜索1](C:\Users\wangyu\Desktop\博客\网易云搜索1.png)

​	通过在网易云音乐进行关键字搜索，并通过F12检测，发现网页给网站https://music.163.com/weapi/cloudsearch/get/web?csrf_token=发送了一条post请求(经测试发现csrf_token=后的数值与登录账号有关)，post的data中包含了params和encSecKey两条数据（因此无需再使用selenium来提取iframe内的数据）：

![网易云搜索2](C:\Users\wangyu\Desktop\博客\网易云搜索2.png)

​	通过检查其preview我们可以发现，其中包含了我们所需要搜索歌曲的信息，如下所示：

![网易云搜索3](C:\Users\wangyu\Desktop\博客\网易云搜索3.png)

​	由于post的data参数为params及encSecKey，因此为了获取preview的json数据，需要对params及encSecKey进行解密，而解密需要获取解密的key，因此，在开发者工具中搜索encSecKey，如图所示：

![网易云搜索4](C:\Users\wangyu\Desktop\博客\网易云搜索4.png)



​	然后点开core开头的文件，通过阅读可以发现加密函数：![网易云搜索5](C:\Users\wangyu\Desktop\博客\网易云搜索5.png)



```
var bVZ7S = window.asrsea(JSON.stringify(i7b), bqN2x(["流泪", "强"]), bqN2x(Wx5C.md), bqN2x(["爱心", "女孩", "惊恐", "大笑"]));         
```

​	i7b为加密前的数据，["流泪", "强"]查函数上方的表可知为“010001”，Wx5C.md根据上方的表可知为Wx5C.md = ["色", "流感", "这边", "弱", "嘴唇", "亲", "开心", "呲牙", "憨笑", "猫", "皱眉", "幽灵", "蛋糕", "发怒", "大哭", "兔子", "星星", "钟情", "牵手", "公鸡", "爱意", "禁止", "狗", "亲亲", "叉", "礼物", "晕", "呆", "生病", "钻石", "拜", "怒", "示爱", "汗", "小鸡", "痛苦", "撇嘴", "惶恐", "口罩", "吐舌", "心碎", "生气", "可爱", "鬼脸", "跳舞", "男孩", "奸笑", "猪", "圈", "便便", "外星", "圣诞"]，查表拼接为"00e0b509f6259df8642dbc35662901477df22677ec152b5ff68ace615bb7b725152b3ab17a876aea8a5aa76d2e417629ec4ee341f56135fccf695280104e0312ecbda92557c93870114af6c9d05c4f7f0c3685b7a46bee255932575cce10b424d813cfe4875d3e82047b97ddef52741d546b8e289dc6935b3ece0462db0a22b8e7"，["爱心", "女孩", "惊恐", "大笑"]同样查表可知为“0CoJUm6Qyw8W8jud”

​	而函数window.asrsea()根据上方function可知为d函数：

```
!function() {
    function a(a) {
        var d, e, b = "abcdefghijklmnopqrstuvwxyzABCDEFGHIJKLMNOPQRSTUVWXYZ0123456789", c = "";
        for (d = 0; a > d; d += 1)
            e = Math.random() * b.length,
            e = Math.floor(e),
            c += b.charAt(e);
        return c
    }
    function b(a, b) {
        var c = CryptoJS.enc.Utf8.parse(b)
          , d = CryptoJS.enc.Utf8.parse("0102030405060708")
          , e = CryptoJS.enc.Utf8.parse(a)
          , f = CryptoJS.AES.encrypt(e, c, {
            iv: d,
            mode: CryptoJS.mode.CBC
        });
        return f.toString()
    }
    function c(a, b, c) {
        var d, e;
        return setMaxDigits(131),
        d = new RSAKeyPair(b,"",c),
        e = encryptedString(d, a)
    }
    function d(d, e, f, g) {
        var h = {}
          , i = a(16);
        return h.encText = b(d, g),
        h.encText = b(h.encText, i),
        h.encSecKey = c(i, e, f),
        h
    }
    function e(a, b, d, e) {
        var f = {};
        return f.encText = c(a + e, b, d),
        f
    }
    window.asrsea = d,
    window.ecnonasr = e
}();
```

​	因此我们可以通过在d函数处设置断点查看post发送的原始数据是什么：

![网易云搜索6](C:\Users\wangyu\Desktop\博客\网易云搜索6.png)

​	具体数据如下：d: "{"hlpretag":"<span class=\"s-fc7\">","hlposttag":"</span>","s":"浅唱","type":"1","offset":"0","total":"true","limit":"30","csrf_token":""}"

因此，搜索不同的信息时仅需修改“s”对应的数值即可，且整个d的数据格式为str，我们仅需将其整体打包为str即可。

## 二、实现加密函数

1. function a(a):

   函数a的主要功能是为了实现从字符串b中随机采样a个不同字符，由于这里调用时默认是16位，因此可以通过以下python程序简单实现：

   ```
   def get_i():
       txt = 'abcdefghijklmnopqrstuvwxyzABCDEFGHIJKLMNOPQRSTUVWXYZ0123456789'
       return ''.join(random.sample(txt, 16))
   ```

2. function b(a, b)：

   函数b的主要功能对post的data以及加密的key进行encode，然后进行偏移量为iv的加密。

   ```
   def AES_encrypt(text, key, iv):
       bs = AES.block_size
       pad2 = lambda s: s + (bs - len(s) % bs) * chr(bs - len(s) % bs)
       encryptor = AES.new(to_16(key), AES.MODE_CBC,to_16(iv))
       encrypt_aes = encryptor.encrypt(str.encode(pad2(text)))
       encrypt_text = str(base64.encodebytes(encrypt_aes), encoding='utf-8')
       return encrypt_text
   ```

3.  function c(a, b, c)：

   函数c的主要功能为了生成encSecKey

   ```
   def RSA_encrypt(text, pubKey, modulus):
       text=text[::-1]
       rs=int(codecs.encode(text.encode('utf-8'), 'hex_codec'), 16) ** int(pubKey, 16) % int(modulus, 16)
       return format(rs, 'x').zfill(256)
   ```

   params参数是通过两次b函数得到。

## 三、生成下载url

​	类似于获取歌单信息，首先需要找到post请求的网站地址，依然是通过F12检查，通过删选信息，可以发现这条post信息中包含了歌曲下载的url。

![网易云搜索7](C:\Users\wangyu\Desktop\博客\网易云搜索7.png)

![网易云搜索8](C:\Users\wangyu\Desktop\博客\网易云搜索8.png)

post所发送的data依然是params和encSecKey，post的目标网站为https://music.163.com/weapi/song/enhance/player/url/v1?csrf_token=，同理我们可以设置断点来检查所post的加密前的data，如图：

![网易云搜索10](C:\Users\wangyu\Desktop\博客\网易云搜索10.png)

data的数据格式：{"ids":str([id]),"level":"standard","encodeType":"aac", "csrf_token": ""}，其中id表示歌曲的id号，level是音乐品质，经我测试标准为standard，较高音质为higher，极高音质没测试出关键词，无损音质关键词为lossless。

另外还发现一个有趣的事情：

![网易云搜索11](C:\Users\wangyu\Desktop\博客\网易云搜索11.png)

![网易云搜索12](C:\Users\wangyu\Desktop\博客\网易云搜索12.png)

对vip歌曲，普通用户是无法发送post请求，因此需要使用vip账号的cookies登录才可以下载vip歌曲。所以说还是支持正版吧。



## 四、完整代码

```python
import urllib.request,os,json
import requests,random
import base64,codecs
from Crypto.Cipher import AES
import pickle

def to_16(key):
    while len(key) % 16 != 0:
        key += '\0'
    return str.encode(key)

def AES_encrypt(text, key, iv):
    bs = AES.block_size
    pad2 = lambda s: s + (bs - len(s) % bs) * chr(bs - len(s) % bs)
    encryptor = AES.new(to_16(key), AES.MODE_CBC,to_16(iv))
    encrypt_aes = encryptor.encrypt(str.encode(pad2(text)))
    encrypt_text = str(base64.encodebytes(encrypt_aes), encoding='utf-8')
    return encrypt_text

def RSA_encrypt(text, pubKey, modulus):
    text=text[::-1]
    rs=int(codecs.encode(text.encode('utf-8'), 'hex_codec'), 16) ** int(pubKey, 16) % int(modulus, 16)
    return format(rs, 'x').zfill(256)

#获取i值的函数，即随机生成长度为16的字符串
def get_i():
    txt = 'abcdefghijklmnopqrstuvwxyzABCDEFGHIJKLMNOPQRSTUVWXYZ0123456789'
    return ''.join(random.sample(txt, 16))

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

class WanYiYun():
    def __init__(self):
        self.url_search='https://music.163.com/weapi/cloudsearch/get/web?csrf_token=' #post地址
        self.song_url='https://music.163.com/weapi/song/enhance/player/url/v1?csrf_token='
        self.g = '0CoJUm6Qyw8W8jud'#buU9L(["爱心", "女孩", "惊恐", "大笑"])的值
        self.b = "010001"#buU9L(["流泪", "强"])的值
        # buU9L(Rg4k.md)的值
        self.c = '00e0b509f6259df8642dbc35662901477df22677ec152b5ff68ace615bb7b725152b3ab17a876aea8a5aa76d2e417629ec4ee341f56135fccf695280104e0312ecbda92557c93870114af6c9d05c4f7f0c3685b7a46bee255932575cce10b424d813cfe4875d3e82047b97ddef52741d546b8e289dc6935b3ece0462db0a22b8e7'
        self.i = get_i()#随机生成长度为16的字符串
        self.iv = "0102030405060708"  # 偏移量
        if not os.path.exists("d:/music"):
            os.mkdir('d:/music')
        self.headers={  'User-Agent':set_user_agent(),
                        'Referer':'https://music.163.com/',
                        'Content-Type':'application/x-www-form-urlencoded'
                        }


    def get_params(self,id):
        #获取加密后的params
        if isinstance(id, int):
            #标准 standard 较高  higher  无损lossless
            encText = {"ids":str([id]),"level":"higher","encodeType":"aac", "csrf_token": ""}
        elif id == None:
            encText = {}
        else:
            encText = {"hlpretag": "<span class=\"s-fc7\">", "hlposttag": "</span>", "s": id, "type": "1", "offset": "0",
                    "total": "true", "limit": "30", "csrf_token": ""}
        encText = json.dumps(encText)
        return AES_encrypt(AES_encrypt(encText,self.g, self.iv), self.i, self.iv)

    def get_encSecKey(self):
        #获取加密后的encSeckey
        return RSA_encrypt(self.i, self.b, self.c)

    def get_search(self, str):
        formdata = {'params': self.get_params(str),
                    'encSecKey': self.get_encSecKey()}
        res = requests.post(self.url_search, data=formdata)
        # 获取歌曲列表的json数据
        song_info = res.json()['result']['songs']
        return song_info

    def get_download_url(self, name):
        music_list = self.get_search(name)
        cookie = pickle.load(open("wyy_cookie.pkl", "rb"))
        cookies = {}
        for c in cookie:
            cookies[c['name']] = c['value']
        cnt = 0
        words = '网易云：\n'
        for music in music_list:
            if cnt == 3:
                break
            cnt = cnt + 1
            music_id = music['id']
            # print(type(music_id))
            music_name = music['name']
            music_author = music['ar'][0]['name']
            music_album = music['al']['name']
            words = words + '歌名:' + music_name + '\n'
            words = words + '歌手:' + music_author + '\n'
            words = words + '专辑名:' + music_album + '\n'
            formdata = {'params':self.get_params(music_id),
                      'encSecKey':self.get_encSecKey()}

            response = requests.post(self.song_url, headers=self.headers, data=formdata, cookies = cookies)
            download_url = json.loads(response.content)["data"][0]["url"]
            if download_url:
                words = words + '下载链接:' + download_url + '\n'
            else:
                words = words + '下载链接:' + '无' + '\n'
        print(words)

if __name__ == '__main__':
    wanyiyun=WanYiYun()
    # name = input("网易云搜索关键词：")
    wanyiyun.get_download_url('许嵩')
```

