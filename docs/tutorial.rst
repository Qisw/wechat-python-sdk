快速上手
=========================

本页内容提供了一些可以快速上手的短小示例代码，并假设你已经安装了 ``wechat-sdk`` 。如果还没有，请返回安装一节进行安装。

注意事项
-------------------------

因为本开发包并不打算接管 HTTP 请求及响应阶段，包括微信服务器 Request 参数的提取、Request Body 的提取、返回 Response 这些操作请自行完成，本页示例中假设开发包所需要的各个参数都已经提取完毕并且最后得到的响应内容会被返回。

**关于字符串中的中文问题，请保证所有在代码中出现的包含汉字的字符串均为 unicode 类型，微信服务器发送过来的 Request Body 不受此限制，请注意下面的代码中关于字符串类型的地方。**

官方接口 - 基本回复使用方法
------------------------------

第一个例子(examples/tutorial_official_1.py)，根据用户发送的内容返回用户发送内容的类型文字说明，比如用户发送了一张图片，则返回“图片”，发送了一段文字，则返回“文字”。

这里附加一个操作，当用户发送的是文字并且内容为“wechat”时返回“^_^”笑脸符号。

::

    # -*- coding: utf-8 -*-

    from wechat_sdk import WechatBasic


    # 下面这些变量均假设已由 Request 中提取完毕
    token = 'WECHAT_TOKEN'  # 你的微信 Token
    signature = 'f24649c76c3f3d81b23c033da95a7a30cb7629cc'  # Request 中 GET 参数 signature
    timestamp = '1406799650'  # Request 中 GET 参数 timestamp
    nonce = '1505845280'  # Request 中 GET 参数 nonce
    # 用户的请求内容 (Request 中的 Body)
    # 请更改 body_text 的内容来测试下面代码的执行情况
    body_text = """
    <xml>
    <ToUserName><![CDATA[touser]]></ToUserName>
    <FromUserName><![CDATA[fromuser]]></FromUserName>
    <CreateTime>1405994593</CreateTime>
    <MsgType><![CDATA[text]]></MsgType>
    <Content><![CDATA[wechat]]></Content>
    <MsgId>6038700799783131222</MsgId>
    </xml>
    """

    # 实例化 wechat
    wechat = WechatBasic(token=token)
    # 对签名进行校验
    if wechat.check_signature(signature=signature, timestamp=timestamp, nonce=nonce):
        # 对 XML 数据进行解析 (必要, 否则不可执行 response_text, response_image 等操作)
        wechat.parse_data(body_text)
        # 获得解析结果, message 为 WechatMessage 对象 (wechat_sdk.messages中定义)
        message = wechat.get_message()

        response = None
        if message.type == 'text':
            if message.content == 'wechat':
                response = wechat.response_text(u'^_^')
            else:
                response = wechat.response_text(u'文字')
        elif message.type == 'image':
            response = wechat.response_text(u'图片')
        else:
            response = wechat.response_text(u'未知')

        # 现在直接将 response 变量内容直接作为 HTTP Response 响应微信服务器即可，此处为了演示返回内容，直接将响应进行输出
        print response

第二个例子(examples/tutorial_official_2.py)，假如用户输入“新闻”，则服务器响应一段预设的多图文

::

    # -*- coding: utf-8 -*-

    from wechat_sdk import WechatBasic


    # 下面这些变量均假设已由 Request 中提取完毕
    token = 'WECHAT_TOKEN'  # 你的微信 Token
    signature = 'f24649c76c3f3d81b23c033da95a7a30cb7629cc'  # Request 中 GET 参数 signature
    timestamp = '1406799650'  # Request 中 GET 参数 timestamp
    nonce = '1505845280'  # Request 中 GET 参数 nonce
    # 用户的请求内容 (Request 中的 Body)
    # 请更改 body_text 的内容来测试下面代码的执行情况
    body_text = """
    <xml>
    <ToUserName><![CDATA[touser]]></ToUserName>
    <FromUserName><![CDATA[fromuser]]></FromUserName>
    <CreateTime>1405994593</CreateTime>
    <MsgType><![CDATA[text]]></MsgType>
    <Content><![CDATA[新闻]]></Content>
    <MsgId>6038700799783131222</MsgId>
    </xml>
    """

    # 实例化 wechat
    wechat = WechatBasic(token=token)
    # 对签名进行校验
    if wechat.check_signature(signature=signature, timestamp=timestamp, nonce=nonce):
        # 对 XML 数据进行解析 (必要, 否则不可执行 response_text, response_image 等操作)
        wechat.parse_data(body_text)
        # 获得解析结果, message 为 WechatMessage 对象 (wechat_sdk.messages中定义)
        message = wechat.get_message()

        response = None
        if message.type == 'text' and message.content == u'新闻':
            response = wechat.response_news([
                {
                    'title': u'第一条新闻标题',
                    'description': u'第一条新闻描述，这条新闻没有预览图',
                    'url': u'http://www.google.com.hk/',
                }, {
                    'title': u'第二条新闻标题, 这条新闻无描述',
                    'picurl': u'http://doraemonext.oss-cn-hangzhou.aliyuncs.com/test/wechat-test.jpg',
                    'url': u'http://www.github.com/',
                }, {
                    'title': u'第三条新闻标题',
                    'description': u'第三条新闻描述',
                    'picurl': u'http://doraemonext.oss-cn-hangzhou.aliyuncs.com/test/wechat-test.jpg',
                    'url': u'http://www.v2ex.com/',
                }
            ])

        # 现在直接将 response 变量内容直接作为 HTTP Response 响应微信服务器即可，此处为了演示返回内容，直接将响应进行输出
        print response

官方接口 - 如何判断消息的类型
---------------------------------

微信服务器发来的POST请求可能会有文字/语音/图片/视频/链接/地理位置/事件，对于如何判定自己通过 ``get_message()`` 获得到的 ``WechatMessage`` 对象类型，这里提供一个示例代码（为了简洁起见，代码中出现的 ``token`` ``signature`` ``timestamp`` ``nonce`` ``body_text`` 均已从 HTTP Request 中提取完毕，具体形式可参照上面的代码，这里不再重复）：

::

    # -*- coding: utf-8 -*-

    from wechat_sdk import WechatBasic
    from wechat_sdk.messages import (
        TextMessage, VoiceMessage, ImageMessage, VideoMessage, LinkMessage, LocationMessage, EventMessage
    )


    # 下面这些变量均假设已由 Request 中提取完毕
    # token, signature, timestamp, nonce, body_text

    # 实例化 wechat
    wechat = WechatBasic(token=token)
    # 对签名进行校验
    if wechat.check_signature(signature=signature, timestamp=timestamp, nonce=nonce):
        # 对 XML 数据进行解析 (必要, 否则不可执行 response_text, response_image 等操作)
        wechat.parse_data(body_text)
        # 获得解析结果, message 为 WechatMessage 对象 (wechat_sdk.messages中定义)
        message = wechat.get_message()

        response = None
        if isinstance(message, TextMessage):
            response = wechat.response_text(content=u'文字信息')
        elif isinstance(message, VoiceMessage):
            response = wechat.response_text(content=u'语音信息')
        elif isinstance(message, ImageMessage):
            response = wechat.response_text(content=u'图片信息')
        elif isinstance(message, VideoMessage):
            response = wechat.response_text(content=u'视频信息')
        elif isinstance(message, LinkMessage):
            response = wechat.response_text(content=u'链接信息')
        elif isinstance(message, LocationMessage):
            response = wechat.response_text(content=u'地理位置信息')
        elif isinstance(message, EventMessage):  # 事件信息
            if message.type == 'subscribe':  # 关注事件(包括普通关注事件和扫描二维码造成的关注事件)
                if message.key and message.ticket:  # 如果 key 和 ticket 均不为空，则是扫描二维码造成的关注事件
                    response = wechat.response_text(content=u'用户尚未关注时的二维码扫描关注事件')
                else:
                    response = wechat.response_text(content=u'普通关注事件')
            elif message.type == 'unsubscribe':
                response = wechat.response_text(content=u'取消关注事件')
            elif message.type == 'scan':
                response = wechat.response_text(content=u'用户已关注时的二维码扫描事件')
            elif message.type == 'location':
                response = wechat.response_text(content=u'上报地理位置事件')
            elif message.type == 'click':
                response = wechat.response_text(content=u'自定义菜单点击事件')
            elif message.type == 'view':
                response = wechat.response_text(content=u'自定义菜单跳转链接事件')
            elif message.type == 'templatesendjobfinish':
                response = wechat.response_text(content=u'模板消息事件')

        # 现在直接将 response 变量内容直接作为 HTTP Response 响应微信服务器即可，此处为了演示返回内容，直接将响应进行输出
        print response

官方接口-在线翻译微信公众号demo， 结合Flask框架
-------------------------
依赖: flask requests wechat-sdk
::
    # coding=utf-8
    
    import sys
    import threading
    reload(sys)
    sys.setdefaultencoding('utf-8')  # 设定默认编码为utf-8 ， 兼容python2.x
    from flask import Flask
    from flask import request
    import time
    import json
    import requests
    from wechat_sdk import WechatBasic
    
    app = Flask(__name__)
    
    token = '你在微信公众号设置的token字符串'
    appid = '你的公众号appid'
    appsecret = '你的公众号appsecret'
    @app.route('/', methods=['GET', 'POST'])
    def index():
        signature = request.args['signature']   # 获取signature
        timestamp = request.args['timestamp']   # 获取timestamp
        nonce = request.args['nonce']           # 获取nonce
        wechat = WechatBasic(token=token, appid=appid, appsecret=appsecret)
        if request.method == 'GET':         # 与微信服务器验证时请求method为GET
            if wechat.check_signature(signature=signature, timestamp=timestamp, nonce=nonce):  # 进行与微信服务器的验证
                return request.args['echostr']          # 验证成功，根据微信开发接口要求原样返回echostr
            else:
                return ""                               # 验证失败，返回空字符串
        elif request.method == 'POST':
            wechat.parse_data(request.data)     
            message = wechat.get_message()      # 解析用户消息
            response = None
            if message.type == 'text':      # 判断消息类型
                if message.content == 'wechat':
                    response = wechat.response_text('^_^')
                else:
                    request_word = message.content  # 获取要翻译的词组
                    try:
                        translate_result = request_translate(request_word)  # 进行翻译，并获取翻译结果
                        response = wechat.response_text(translate_result)    # 将结果转换成微信接口定义的格式
                    except Exception:
                        response = wechat.response_text('Something wrong')   # 不能正确翻译
            else:
                response = wechat.response_text('词组格式错误') 
        return response     # 向微信服务器返回结果
    
    def request_translate(word):
        """
        :@word str 要翻译的词组
        :@retrun str 翻译结果
        翻译使用有道提供的接口，可以免费申请
        """
        # param为传送给有道翻译api的数据， key和keyform在申请成功后的邮件内容中获取
        param = {'keyfrom' : 'easytranslate', 'key':'申请到的有道api的key参数值', 'type':'data', 'doctype':'json', 'version':'1.1', 'q':word}
        url = r'http://fanyi.youdao.com/openapi.do'  # 有道翻译api url地址
        resp = requests.get(url, params=param)       # 利用requests进行请求
        tmp_str = resp.json()['query'] + ": \n"
        resp_json = resp.json()['basic']            # 获取返回的翻译结果
        for key in resp_json.keys():
            # 取出想要的数据，因为有道翻译api返回的翻译结果有很多
            # 这里只取其中感兴趣的一部分，并将结果拼接存放在tmp_str中
            if "phonetic" in key:
                continue
            tmp = key + ": "
            if isinstance(resp_json[key], list):
                for i in resp_json[key]:
                    tmp += "".join(i) + "\n"
            else:
                tmp += resp_json[key] + "\n"
            tmp_str += tmp
        tmp_str += r'<a href="http://fanyi.baidu.com/?aldtype=16047#en/zh/{}">read more</a>'.format(word)
        return tmp_str  # 返回翻译结果字符串
    
    if __name__ == "__main__":
        app.run(debug=True, port=5000, host='0.0.0.0')  # 运行服务


非官方接口 - 基本用法
-------------------------

第一个例子(examples/tutorial_unofficial_1.py)，展示了几个直接获取信息的函数的用法，至于具体的返回值所包含的内容，请查看 ``WechatExt`` 文档

::

    # -*- coding: utf-8

    import json

    from wechat_sdk import WechatExt


    wechat = WechatExt(username='username', password='password')

    # 获取未分组中所有的用户成员
    user_list = wechat.get_user_list()
    print user_list
    print '==================================='

    # 获取分组列表
    group_list = wechat.get_group_list()
    print group_list
    print '==================================='

    # 获取图文信息列表
    news_list = wechat.get_news_list(page=0, pagesize=15)
    print news_list
    print '==================================='

    # 获取与最新一条消息用户的对话内容
    user_info_json = wechat.get_top_message()
    user_info = json.loads(user_info_json)
    print wechat.get_dialog_message(fakeid=user_info['msg_item'][0]['fakeid'])
