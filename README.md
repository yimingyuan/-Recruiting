[首页](https://tech.bytedance.net/)[文章](https://tech.bytedance.net/articles)[视频](https://tech.bytedance.net/videos)[问答](https://tech.bytedance.net/questions)[活动](https://tech.bytedance.net/activity)[新人培训](https://tech.bytedance.net/bootcamp/last)[团队号](https://tech.bytedance.net/teams)更多



![avatar](https://s3-fs.pstatp.com/static-resource/v1/186acd3a-2be9-4919-acfd-66db1442f89g~?image_size=72x72&cut_type=&quality=&format=webp&sticker_format=.webp)

![zh-CN](https://cdn-tos-cn.bytedance.net/obj/archi/tech/assets/i18n-zh.46becd91.svg)



**关于作者**

![作者头像](https://s1-imfile.feishucdn.com/static-resource/v1/a235bcf5-a3c0-40f9-9fb3-cae3b6302f9g~?image_size=96x96&cut_type=&quality=&format=webp&sticker_format=.webp)

楼远洋![level1.png](https://cdn-tos-cn.bytedance.net/obj/archi/tech/assets/lv1.5a3f50be.png)

3人关注 · 21获赞

关注

经验值：59

所属团队号：

![img](https://p-bytetech.bytedance.net/tos-cn-i-vz0z6vmpra/4ef6d87044f747e1b55bfd950fe7ec44~tplv-vz0z6vmpra-image.image)Lark Messenger 团队

**收录专栏**

Lark Messenger 团队专栏

![img](https://cdn-tos-cn.bytedance.net/obj/archi/tech/assets/followers-icon.b3320e60.svg) 321![img](https://cdn-tos-cn.bytedance.net/obj/archi/tech/assets/articles-icon.82797f08.svg) 158

**相关推荐**

Abase2：字节跳动新一代高可用NoSQL数据库

吴振宇

555

50

财经支付-计费业务分享

李政

43

4

技术专业力视频好课，随时随地学不停！

丁银萍

3145

131

那些年，我们背过的面试题

于景洋

3246

164

[![banner](https://tech-proxy.bytedance.net/tos/images/1652236611011_4bfed532613d1d10dd06ebb75f1be476.gif)](https://tech.bytedance.net/articles/7095649053966336008)[![banner](https://tech-proxy.bytedance.net/tos/images/1652424062294_2408f50ffa216e8ff021938d4c7767ad.jpg)](https://live.juejin.cn/4354/juejinyetan003/?source=ByteTech)

如何实现一个牛客网定时发帖机器人？

![楼](https://s1-imfile.feishucdn.com/static-resource/v1/a235bcf5-a3c0-40f9-9fb3-cae3b6302f9g~?image_size=40x40&cut_type=&quality=&format=webp&sticker_format=.webp)[楼远洋](https://tech.bytedance.net/articles/6941650074782924831?from=lark_all_search#)发表于[Lark Messenger 团队专栏](https://tech.bytedance.net/article/series/detail/6960652819854688287)

2021-03-20

Python爬虫Lark messenger

招聘是个流程很长，转化率相对较低的工作，我们需要用尽量少的人力来达成更多的Offer。本机器人可以用尽量少的人力实现牛客网的自动发帖，被动接受简历。

## 1. 登录

[牛客网登录链接](https://www.nowcoder.com/login?callBack=https%3A%2F%2Fwww.nowcoder.com%2Fprofile%2F5635415)

可以看到牛客网的登录无需验证码，这对爬虫非常友好，我们直接使用测试手机号和密码尝试登录，当然，登录会失败，不过我们可以看到登录接口的请求：

![image-20210318202819291](https://pic-bed-louyy.oss-cn-beijing.aliyuncs.com/image-20210318202819291.png)

关注右下角的请求，可以看到三个参数：账号，是否自动登录，加密后的密码。显然，只要知道cipherPwd的加密过程，我们就可以复刻这个请求，从而拿到登录后的session。

------

可以肯定的是，加密过程是在前端完成的。使用元素选择器选中**立即登录**按钮，看看在点击登录后发生了什么。

![image-20210318203436940](https://pic-bed-louyy.oss-cn-beijing.aliyuncs.com/image-20210318203436940.png)

这里没有显示绑定`onClick`事件，继续搜一下id，发现在`main.entry.js`中搜到了。虽然js文件被混淆压缩，我们仍然可以通过chrome的format工具查看简单的逻辑

![image-20210318204250616](https://pic-bed-louyy.oss-cn-beijing.aliyuncs.com/image-20210318204250616.png)

格式化后再次搜索发现绑定的事件

```javascript
initSubmit: function() {
                u("#jsLoginBtn").on("click", h.bind(this.submit, this)),
                u("div.input-section form").on("submit", h.bind(this.submit, this))
            },
```

登录按钮点击后，执行了submit函数，看看submit都有啥

```javascript
            submit: function(i) {
                i.preventDefault();
                var t = this
                  , n = u("#jsLoginBtn");
                t.clear(),
                t.check((function(i) {
                    i && !f.clear(n) && b.login({
                        body: t.val(!0),
                        captcha: {
                            type: S.captcha.login
                        },
                        call: function(i) {
                            return t.jump()
                        },
                        error: function(i) {
                            var n = 4 === i.code;
                            n && t.cpn.pwd.setErrorTips(i.msg || "密码不正确，请重试！"),
                            !n && t.cpn.account.setErrorTips(i.msg)
                        },
                        always: function() {
                            return f.clear(n)
                        }
                    })
                }
                ))
            },
```

在login中看到了 call、error、 always，一定的经验告诉我们，这个地方应该就是发起ajax的代码段，那么对应的需要关注下`body: t.val(!0)` 干了什么，观察 val 函数的返回

![image-20210318204902126](https://pic-bed-louyy.oss-cn-beijing.aliyuncs.com/image-20210318204902126.png)

可以看到，这里调用了encrypt对密码原文加了密赋值给了n返回，进一步在当前文件查找encrypt

```js
        encrypt: function(i) {
            var t = new window.NCJSEncrypt;
            return t.setPublicKey(u("#jsNCPublicKey").html() || ""),
            t.encrypt(u.trim(i))
        }
```

可以看到，这里从DOM中取了一个元素的内容，回Elements选项卡看下内容

![image-20210318205318324](https://pic-bed-louyy.oss-cn-beijing.aliyuncs.com/image-20210318205318324.png)

根据id命名，这个部分应该就是加密用的公钥了，可见牛客用了非对称加密算法，那么常用的非对称加密算法有：RSA、ECC（移动设备用）、Diffie-Hellman、El Gamal、DSA（数字签名用）。目前使用最广的是RSA，在Source 里面再搜一下`RSA` 或者 `publicKey`等关键词，可以搜索到

![image-20210320190347575](https://pic-bed-louyy.oss-cn-beijing.aliyuncs.com/image-20210320190347575.png)

到这里可以写代码用RSA加密尝试一下了。示例代码：

```python
import base64

import requests
from Crypto.PublicKey import RSA
from Crypto.Cipher import PKCS1_v1_5 as Cipher_pkcs1_v1_5

from nowcoderbot import rsa_public_key
from nowcoderbot.login.NCLogin import NCLogin

session = requests.session()

rsa_public_key = """-----BEGIN PUBLIC KEY-----
MIGfMA0GCSqGSIb3DQEBAQUAA4GNADCBiQKBgQCsAiBmrht+wJFrQSMlMjppAYniUSvyek62PDJg
g/+dWoODaPEkaGTXpGmSuRyn3RC2xWLPKZIxvYc9Pk+/QDnmsNxaY/3T3btZmgJsKPw4F7YCfRY/
fKk/lahvumwnohr8cFY9lVgAz80caWcP9SZijQ9MaXgW3GkMkuWyhcoJpwIDAQAB
-----END PUBLIC KEY-----"""

class NCPasswordLogin(NCLogin):

    def __init__(self, account, pwd):
        self.account = account
        self.pwd = pwd
        self.cipher_text = base64.b64encode(
            Cipher_pkcs1_v1_5.new(RSA.importKey(rsa_public_key)).encrypt(bytes(self.pwd, encoding="utf8")))

    def do(self, session=None) -> requests.Session:
        if not session:
            session = requests.session()

        session.get("https://www.nowcoder.com/login")
        session.post(url="https://www.nowcoder.com/nccommon/login/do?token=", data={
            "email": self.account,
            "remember": "true",
            "cipherPwd": self.cipher_text
        })

        return session


if __name__ == '__main__':
    session = NCPasswordLogin("******", "*********").do()
    print(session.get("https://www.nowcoder.com/").text)
```

登录成功

![image-20210320190942200](https://pic-bed-louyy.oss-cn-beijing.aliyuncs.com/image-20210320190942200.png)

## 2. 发帖

[发帖地址](https://www.nowcoder.com/discuss/v2/post?type=7#discuss)

由于发帖后页面会重定向，network的请求会重置从而无法及时捕捉到相关请求（经提醒，可以勾选preserve log 选项），一种做法是使用Charles抓包，或者连续发两次相同内容的帖子，第二次接口返回失败后页面不会重定向，从而捕捉到了发帖接口

![image-20210320192811484](https://pic-bed-louyy.oss-cn-beijing.aliyuncs.com/image-20210320192811484.png)

需要关注的几个参数是：

- `title` 标题
- `content` HTML内容
- `disscussType` 板块，7表示招聘信息，一般发内推信息都使用7
- `tag` 标签，多个标签容易被搜索到
- `subjectIds` 话题，265是 #字节跳动#
- `hasSubject` 是否包含话题

经测试 `mdContent`这个字段即使在MD编辑器下发送请求也不生效，都以content字段为准。但是我们需要通过机器人实现帖子可配置化发帖的话，md格式比html灵活不少，所以写脚本时，需要把md转成html，`markdown`可以实现

```python
def markdown(text, **kwargs):
    """Convert a markdown string to HTML and return HTML as a unicode string.

    This is a shortcut function for `Markdown` class to cover the most
    basic use case.  It initializes an instance of Markdown, loads the
    necessary extensions and runs the parser on the given text.

    Keyword arguments:

    * text: Markdown formatted text as Unicode or ASCII string.
    * Any arguments accepted by the Markdown class.

    Returns: An HTML document as a string.

    """
    md = Markdown(**kwargs)
    return md.convert(text)
```

示例代码（基于已登录session）

```python
import markdown

from nowcoderbot import post_header
from nowcoderbot.login.NCPasswordLogin import NCPasswordLogin


def process_post(session):
    with open(r'post/intern.md', 'r') as f:
        print(session.post("https://www.nowcoder.com/discuss/create?token=", headers=post_header, data={
            "title": "字节跳动实习生内推",
            "content": markdown.markdown(f.read()),
            "mdContent": "",
            "contentType": "1",
            "type": "7",
            "tags": "861, 827",
            "timed": "NaN",
            "subjectIds": "265",
            "hasSubject": "true",
        }).text)



if __name__ == '__main__':
    session = NCPasswordLogin("******", "******").do()
    process_post(session)
```

## 3. 其它

- 配合crontab就可以实现定时自动发帖了，脚本地址[Github](https://github.com/yylou15/nowcoder_bot), 里面同时包含了牛客扫二维码登录的逻辑，但扫码操作不够自动化，本文不展开叙述
- 由于牛客对帖子重复度有校验，短时间内重复发帖会触发系统删帖/禁言，建议间隔在3-4小时
- 由于目前没有足够的账号资源实现大规模自动顶贴、收藏、回复等操作，如果你愿意共享账号，可以与我联系

284

12

2

4



\- 12人点赞 -

![王枞](https://s1-imfile.feishucdn.com/static-resource/v1/0f1a831d-0046-4116-9c51-aaa2575ff63g~?image_size=noop&cut_type=&quality=&format=png&sticker_format=.webp)

![崔贵林](https://s1-imfile.feishucdn.com/static-resource/v1/9d244e30-d731-4b9d-96fd-bf0f8a8e5d4g~?image_size=noop&cut_type=&quality=&format=png&sticker_format=.webp)

![程成](https://s1-imfile.feishucdn.com/static-resource/v1/d13f85a5-8334-4c4a-a81b-7851447c841g~?image_size=noop&cut_type=&quality=&format=png&sticker_format=.webp)

![韩绍帅](https://s1-imfile.feishucdn.com/static-resource/v1/v2_d267cea3-d828-4b2e-ab97-58a763019b6g~?image_size=noop&cut_type=&quality=&format=png&sticker_format=.webp)

![贾维娣](https://s1-imfile.feishucdn.com/static-resource/v1/v2_d46e8aab-a2db-47b7-bad5-0413e9c6e70g~?image_size=noop&cut_type=&quality=&format=png&sticker_format=.webp)

![毛程炜](https://s1-imfile.feishucdn.com/static-resource/v1/v2_7eed3591-777a-48f2-b1f6-acc7c7d7614g~?image_size=noop&cut_type=&quality=&format=png&sticker_format=.webp)



取消点赞评论收藏

![袁](https://s3-fs.pstatp.com/static-resource/v1/186acd3a-2be9-4919-acfd-66db1442f89g~?image_size=72x72&cut_type=&quality=&format=webp&sticker_format=.webp)

 

图片

@提及

提交评论

评论2

![龙斌](https://s1-imfile.feishucdn.com/static-resource/v1/dad60017c748276cff57~?image_size=64x64&cut_type=&quality=&format=webp&sticker_format=.webp)

龙斌2021-03-22 10:21

"network的请求会重置从而无法及时捕捉到相关请求" 这个问题是不是开启 network 的 preserve log 就好了

0回复

![楼远洋](https://s1-imfile.feishucdn.com/static-resource/v1/a235bcf5-a3c0-40f9-9fb3-cae3b6302f9g~?image_size=64x64&cut_type=&quality=&format=webp&sticker_format=.webp)

楼远洋2021-03-22 10:42

回复@龙斌：学到了

0回复

\1. 登录

\2. 发帖

\3. 其它

评论区
