
https://github.com/Evil0ctal/Douyin_TikTok_Download_API/issues/529# douyin-watermark
抖音无水印

```
import re
import requests
headers = {
    'User-Agent': 'Mozilla/5.0 (iPhone; CPU iPhone OS 15_0 like Mac OS X) AppleWebKit/605.1.15 (KHTML, like Gecko) Version/15.0 Mobile/15E148 Safari/604.1'
}

def download_douyin_video(url):
    # 获取最终重定向URL
    res = requests.get(url, allow_redirects=True, headers=-headers)
    vid = re.findall(r'/video/(\d+)', res.url)[0]  # 匹配数字ID
    
    # 保存HTML内容
    with open('douyin_page.html', 'w', encoding='utf-8') as f:
        f.write(res.text)
    
    # 打印关键信息
    print(f"最终URL: {res.url}")
    print(f"视频ID: {vid}")
        
if __name__ == "__main__":
    short_url = "https://v.douyin.com/HO1SGREMIQU/"
    video_url = download_douyin_video(short_url)
    """
```    
    

https://github.com/yf8899/yf8899.github.io/issues/1
    https://github.com/MCQTSS/DouYinVideoURL/issues/1

从抖音app分享视频，复制链接是一个`https://v.douyin.com/HO1SGREMIQU` 链接，浏览器再打开他，会跳转到
    `https://www.douyin.com/video/7448816576188419378?previous_page=app_code_link`
    
    ```
    最终URL: https://www.douyin.com/video/7448816576188419378?previous_page=app_code_link
    视频ID: 7448816576188419378
    ```
    
    其中链接的数字`7448816576188419378`就是视频id
    ↑上面是没有ua，也就是没有headers时候的requests最终url，保存的html也很短，完全没有视频内容，
    
    添加headers再请求，最终url就是：
    
    ```
    ~ $ python /storage/emulated/0/抖音.pyy
    最终URL: https://www.iesdouyin.com/share/video/7448816576188419378/?region=CN&mid=7448816801238010674&u_code=13bb.....................
    视频ID: 7448816576188419378
    ```
    
    https://www.iesdouyin.com/web/api/v2/aweme/iteminfo/?item_ids=
    
    上面的链接填入视频id，打开会是空白页，所以上面的方案不行。
    
    需要的是下面的方法：
    直接在视频网页链接的html里面搜索，
    找到
    `"cha_list":null,"video":{"play_addr":{"uri":"v0200fg10000ctfnr0nog65ml1ek1vpg","url_list":["https:\u002F\u002Faweme.snssdk.com\u002Faweme\u002Fv1\u002Fplaywm\u002F?line=0&logo_name=aweme_diversion_search&ratio=720p&video_id=v0200fg10000ctfnr0nog65ml1ek1vpg"]},"cover":{"uri":"tos-cn-i-dy\u002F40bc8591414b4f078a2395a88e127a6b","url_list":["https:\u002F\u002Fp3-sign.douyinpic. ……`
    其中
    https://aweme.snssdk.com/aweme/v1/play/?line=0&logo_name=aweme_diversion_search&ratio=720p&video_id=v0200fg10000ctfnr0nog65ml1ek1vpg 上述链接把playwm变为play就是无水印链接
    
    其中这个json在
    ```
    </head><body><div id="root"><div class="container"><div class="video-container horizontal-video"><img src="https://p3-sign.douyinpic.com/tos-cn-i-dy/40bc8591414b4f078a2395a88e127a6b~tplv-dy-resize-walign-adapt-aq:540:q75.webp?lk3s=138a59ce&amp;x-expires=1749186000&amp;x-signature=9tGX%2FS5XY0MpJEN6KJk8NJQGGc4%3D&amp;from=327834062&amp;s=PackSourceEnum_DOUYIN_REFLOW&amp;se=false&amp;sc=cover&amp;biz_tag=aweme_video&amp;l=20250523130934C2DE67B1C2BC3008E993" class="poster" loading="eager"/><div class="video-msg-container"></div></div></div><script nonce="E8qAYt-OWbzVVB0wZO6bx" async="">_ROUTER_DATA = {"loaderData":{"video_layout":null,"video_(id)\u002Fpage":{"ua":"Mozilla\u002F5.0 (iPhone; CPU iPhone OS 15_0 like Mac OS X) AppleWebKit\u002F605.1.15 (KHTML, like Gecko) Version\u002F15.0 Mobile\u002F15E148 Safari\u002F604.1","isSpider":false,"webId":"7507503860493190668","query":{"region":"CN", ………
    
    ```
    
    他在html中上面这一行
    
    
    """
    
    
    
    
```
import re
import json
import requests

headers = {
    'User-Agent': 'Mozilla/5.0 (iPhone; CPU iPhone OS 15_0 like Mac OS X) AppleWebKit/605.1.15 (KHTML, like Gecko) Version/15.0 Mobile/15E148 Safari/604.1'
}

def find_play_addr(data, path=""):
    """递归查找所有play_addr"""
    results = []
    if isinstance(data, dict):
        if 'play_addr' in data:
            results.append({
                'path': f"{path}['play_addr']",
                'data': data['play_addr']
            })
        for key, value in data.items():
            results.extend(find_play_addr(value, f"{path}['{key}']"))
    elif isinstance(data, list):
        for i, item in enumerate(data):
            results.extend(find_play_addr(item, f"{path}[{i}]"))
    return results

def download_douyin_video(url):
    # 获取最终重定向URL
    res = requests.get(url, allow_redirects=True, headers=headers)
    final_url = res.url
    vid = re.findall(r'/video/(\d+)', final_url)[0]
    
    print(f"最终URL: {final_url}")
    print(f"视频ID: {vid}")
    
    # 在HTML中查找视频信息
    html_content = res.text
    
    # 查找JSON数据
    json_pattern = r'_ROUTER_DATA\s*=\s*({.+?})\s*;'
    match = re.search(json_pattern, html_content)
    
    if not match:
        print("未找到JSON数据")
        return None

    try:
        json_data = json.loads(match.group(1))
        
        # 查找所有可能的play_addr
        play_addrs = find_play_addr(json_data)
        
        if not play_addrs:
            print("未找到play_addr数据")
            return None
            
        print("\n找到所有视频地址:")
        for i, addr in enumerate(play_addrs, 1):
            print(f"\n选项 {i}:")
            print(f"路径: {addr['path']}")
            print("URL列表:")
            for url in addr['data'].get('url_list', []):
                print(url)
        
        # 默认选择第一个找到的play_addr
        selected = play_addrs[0]['data']
        watermark_urls = selected.get('url_list', [])
        
        if not watermark_urls:
            print("没有可用的视频URL")
            return None
            
        # 生成无水印URL（通用方法）
        no_watermark_url = watermark_urls[0].replace('playwm', 'play')
        print(f"\n最终选择的无水印URL: {no_watermark_url}")
        
        return no_watermark_url
        
    except Exception as e:
        print(f"解析出错: {str(e)}")
        return None

if __name__ == "__main__":
    # 示例用法
    short_url = "https://v.douyin.com/HO1SGREMIQU/"
    video_url = download_douyin_video(short_url)
    
    if video_url:
        print("\n下载命令:")
        print(f"curl '{video_url}' -H 'Referer: https://www.douyin.com/' -o douyin_video.mp4")
```
浏览器打开
https://aweme.snssdk.com/aweme/v1/play/?line=0&logo_name=aweme_diversion_search&ratio=720p&video_id=v0200fg10000ctfnr0nog65ml1ek1vpg 会跳转
https://v26-hl-cold.douyinvod.com/62969fb5c0f4bf0d9a72cc8a2694267d/6830457b/video/tos/cn/tos-cn-ve-15/o8ff7vm4OQEUBXBahCvLiCsIIUGjosADFAeXSX/?a=1128&ch=0&cr=0&dr=0&cd=0%7C0%7C0%7C0&cv=1&br=1117&bt=1117&cs=0&ds=3&ft=Bach4VVywSyRKJ8Pmo~ySqTeaApZSxNUvrKqR4krmo0g3cI&mime_type=video_mp4&qs=0&rc=Mzg5ZTw5NmczNDs4OTs2OUBpanluNHA5cjNxdzMzNGkzM0BjLzFfXjJiNjAxMjNeLmI1YSNtLmJpMmRrb2NgLS1kLS9zcw%3D%3D&btag=c0010e000b8001&cquery=100y&dy_q=1747989601&feature_id=aa7df520beeae8e397df15f38df0454c&l=20250523164001832C9501572E670B2C54
并且播放视频，
这个才是真正链接


如果直接curl，会得到一个html文件
<a href="https://v26-hl-cold.douyinvod.com/cdced462fc5a77b08b0245ad2eb96ced/6830456e/video/tos/cn/tos-cn-ve-15/o8ff7vm4OQEUBXBahCvLiCsIIUGjosADFAeXSX/?a=1128&amp;ch=0&amp;cr=0&amp;dr=0&amp;cd=0%7C0%7C0%7C0&amp;cv=1&amp;br=1117&amp;bt=1117&amp;cs=0&amp;ds=3&amp;ft=Bach4VVywSyRKJ8Pmo~ySqTeaAp~SxNUvrKqR4krmo0g3cI&amp;mime_type=video_mp4&amp;qs=0&amp;rc=Mzg5ZTw5NmczNDs4OTs2OUBpanluNHA5cjNxdzMzNGkzM0BjLzFfXjJiNjAxMjNeLmI1YSNtLmJpMmRrb2NgLS1kLS9zcw%3D%3D&amp;btag=c0010e000b8001&amp;cquery=100y&amp;dy_q=1747989588&amp;feature_id=aa7df520beeae8e397df15f38df0454c&amp;l=20250523163948B5D917518AD9AC0B297B">Found</a>.     
    
```
import re
import json
import requests

headers = {
    'User-Agent': 'Mozilla/5.0 (iPhone; CPU iPhone OS 15_0 like Mac OS X) AppleWebKit/605.1.15 (KHTML, like Gecko) Version/15.0 Mobile/15E148 Safari/604.1',
    'Referer': 'https://www.douyin.com/'
}

def get_final_video_url(initial_url):
    """获取最终的视频URL（处理重定向）"""
    session = requests.Session()
    session.headers.update(headers)
    
    # 禁止自动重定向，我们需要手动处理
    resp = session.get(initial_url, allow_redirects=False)
    
    if resp.status_code == 302:
        # 获取重定向URL
        redirect_url = resp.headers.get('Location')
        if redirect_url:
            print(f"重定向到: {redirect_url}")
            return redirect_url
    
    # 如果没有直接重定向，尝试从HTML中提取
    match = re.search(r'<a href="(https?://[^"]+)"', resp.text)
    if match:
        print(f"从HTML中提取到URL: {match.group(1)}")
        return match.group(1)
    
    print("无法获取最终视频URL")
    return None

def download_douyin_video(url):
    # 获取最终重定向URL
    res = requests.get(url, allow_redirects=True, headers=headers)
    final_url = res.url
    vid = re.findall(r'/video/(\d+)', final_url)[0]
    
    print(f"最终页面URL: {final_url}")
    print(f"视频ID: {vid}")
    
    # 在HTML中查找视频信息
    html_content = res.text
    
    # 查找JSON数据
    json_pattern = r'_ROUTER_DATA\s*=\s*({.+?})\s*;'
    match = re.search(json_pattern, html_content)
    
    if not match:
        print("未找到JSON数据")
        return None

    try:
        json_data = json.loads(match.group(1))
        
        # 查找所有可能的play_addr
        play_addrs = find_play_addr(json_data)
        
        if not play_addrs:
            print("未找到play_addr数据")
            return None
            
        print("\n找到所有视频地址:")
        for i, addr in enumerate(play_addrs, 1):
            print(f"\n选项 {i}:")
            print(f"路径: {addr['path']}")
            print("URL列表:")
            for url in addr['data'].get('url_list', []):
                print(url)
        
        # 选择第一个找到的play_addr
        selected = play_addrs[0]['data']
        watermark_urls = selected.get('url_list', [])
        
        if not watermark_urls:
            print("没有可用的视频URL")
            return None
            
        # 生成无水印URL
        no_watermark_url = watermark_urls[0].replace('playwm', 'play')
        print(f"\n获取无水印URL: {no_watermark_url}")
        
        # 获取最终视频URL
        final_video_url = get_final_video_url(no_watermark_url)
        if not final_video_url:
            return None
            
        print(f"\n最终视频URL: {final_video_url}")
        return final_video_url
        
    except Exception as e:
        print(f"解析出错: {str(e)}")
        return None

def find_play_addr(data, path=""):
    """递归查找所有play_addr"""
    results = []
    if isinstance(data, dict):
        if 'play_addr' in data:
            results.append({
                'path': f"{path}['play_addr']",
                'data': data['play_addr']
            })
        for key, value in data.items():
            results.extend(find_play_addr(value, f"{path}['{key}']"))
    elif isinstance(data, list):
        for i, item in enumerate(data):
            results.extend(find_play_addr(item, f"{path}[{i}]"))
    return results

if __name__ == "__main__":
    # 示例用法
    short_url = "https://v.douyin.com/HO1SGREMIQU/"
    video_url = download_douyin_video(short_url)
    
    if video_url:
        print("\n下载命令:")
        print(f"curl '{video_url}' -H 'Referer: https://www.douyin.com/' -o douyin_video.mp4")
        
        # 或者直接下载
        try:
            print("\n开始下载视频...")
            video_data = requests.get(video_url, headers=headers)
            with open('douyin_video.mp4', 'wb') as f:
                f.write(video_data.content)
            print("视频下载完成!")
        except Exception as e:
            print(f"下载失败: {str(e)}")
```        


# 最终代码:

```
import re
import json
import requests

def get_final_video_url(short_url):
    # 获取页面内容
    html = requests.get(short_url, headers={'User-Agent': 'iPhone'}).text
    
    # 提取JSON数据
    json_data = json.loads(re.search(r'_ROUTER_DATA\s*=\s*({.+?})\s*;', html).group(1))
    
    # 查找play_addr
    stack = [json_data]
    while stack:
        data = stack.pop()
        if isinstance(data, dict):
            if 'play_addr' in data:
                url = data['play_addr']['url_list'][0].replace('playwm', 'play')
                print("找到视频URL:", url)
                final_url = requests.get(url, allow_redirects=False).headers['Location']
                print("最终重定向URL:", final_url)
                return final_url
            stack.extend(data.values())
        elif isinstance(data, list):
            stack.extend(data)
    
    return None

# 使用示例
print(get_final_video_url("https://v.douyin.com/HO1SGREMIQU/"))
```

使用curl -L 可以直接下载重定向后的链接，不用再使用较长的视频cdn链接：

```
import re
import json
import requests

def extract_douyin_url(text):
    """从文本中提取抖音短链接"""
    match = re.search(r'https?://v\.douyin\.com/[a-zA-Z0-9_\-/]+', text)
    return match.group(0) if match else None

def get_douyin_curl(input_text, ratio='1080p'):
    """获取抖音视频下载的curl命令"""
    # 1. 提取纯净URL
    clean_url = extract_douyin_url(input_text)
    if not clean_url:
        print("错误：未找到有效的抖音链接")
        return

    print(f"提取的抖音链接: {clean_url}")

    # 2. 获取页面内容
    try:
        html = requests.get(
            clean_url, 
            headers={'User-Agent': 'Mozilla/5.0 (iPhone; CPU iPhone OS 15_0 like Mac OS X)'},
            allow_redirects=True
        ).text
    except Exception as e:
        print(f"请求失败: {e}")
        return

    # 3. 提取JSON数据
    try:
        json_data = json.loads(re.search(r'_ROUTER_DATA\s*=\s*({.+?})\s*;', html).group(1))
    except:
        print("解析JSON失败")
        return

    # 4. 查找视频URI
    stack = [json_data]
    while stack:
        data = stack.pop()
        if isinstance(data, dict):
            if 'video' in data and 'play_addr' in data['video']:
                video_uri = data['video']['play_addr']['uri']
                play_url = f"https://aweme.snssdk.com/aweme/v1/play/?video_id={video_uri}&ratio={ratio}"
                print(f"\n视频ID: {video_uri}")
                print("使用此命令下载（自动处理重定向）：")
                print(f"curl -L '{play_url}' -H 'Referer: https://www.douyin.com/' -o douyin_video.mp4")
                return
            stack.extend(data.values())
        elif isinstance(data, list):
            stack.extend(data)

    print("未找到视频信息")

# 使用示例
input1 = "https://v.douyin.com/HO1SGREMIQU/"
input2 = "2.58 复制打开抖音，看看【会做甜品的小云南的作品】# 菏泽一中# 菏泽高中# 恰巴塔# 学校食堂# ... https://v.douyin.com/-UqU0hdTOh8/ UlP:/ 10/12 n@D.hB"

get_douyin_curl(input1)  # 处理纯URL
get_douyin_curl(input2)  # 处理包含URL的文本
```

输出打印：`curl -L 'https://aweme.snssdk.com/aweme/v1/play/?video_id=v0200fg10000d0o5isvog65j30ug44mg&ratio=1080p' -H 'Referer: https://www.douyin.com/' -o douyin_video.mp4`

其中&ratio=1080p默认720p可换1080p


https://douyin.wtf/api/douyin/web/fetch_one_video?aweme_id=7507166845179415845

搜索2160 关键词 找到：

`bit_rate":[{"FPS":29,"HDR_bit":"","HDR_type":"","bit_rate":2609707,"format":"mp4","gear_name":"adapt_lowest_4_1","is_bytevc1":1,"is_h265":1,"play_addr":{"data_size":376034063,"file_cs":"c:0-947869-74b3|a:v0200fg10000d0nd0vnog65s0g9k36kg","file_hash":"74cabc68c425c171d8592aeb20c266d7","height":2160,"uri":"v0200fg10000d0nd0vnog65s0g9k36kg","url_key":"v0200fg10000d0nd0vnog65s0g9k36kg_bytevc1_4k_2609707","url_list":["https://v5-dy-o-abtest.zjcdn.com/99a816b1ed5127bd9d536d8784304a44/68331dac/video/tos/cn/tos-cn-ve-15/oEBsvABw2CVYNiE0QFivABg0mn6ngpIBemB8Af/?a=6383&ch=26&cr=3&dr=0&lr=all&cd=0%7C0%7C0%7C3&cv=1&br=2548&bt=2548&cs=2&ds=10&ft=CZdgCYlIDyjNNRVQ9wTkQTyhd._33cAg3-ApQX&mime_type=video_mp4&qs=15&rc=aDs5aTtnZjo5ODQ4aDVlZ0BpajluanY5cnlnMzMzNGkzM0AxXmAtXzReNTYxMmFfM15hYSNoMDYtMmRrLWthLS1kLS9zcw%3D%3D&btag=80000e00038000&cquery=100o_100w_100B_100H_100K&dy_q=1748168444&feature_id=10cf95ef75b4f3e7eac623e4ea0ea691&l=202505251820441DD91332211A56296233","https://v5-dy-o-abtest.zjcdn.com/99a816b1ed5127bd9d536d8784304a44/68331dac/video/tos/cn/tos-cn-ve-15/oEBsvABw2CVYNiE0QFivABg0mn6ngpIBemB8Af/?a=6383&ch=26&cr=3&dr=0&lr=all&cd=0%7C0%7C0%7C3&cv=1&br=2548&bt=2548&cs=2&ds=10&ft=CZdgCYlIDyjNNRVQ9wTkQTyhd._33cAg3-ApQX&mime_type=video_mp4&qs=15&rc=aDs5aTtnZjo5ODQ4aDVlZ0BpajluanY5cnlnMzMzNGkzM0AxXmAtXzReNTYxMmFfM15hYSNoMDYtMmRrLWthLS1kLS9zcw%3D%3D&btag=80000e00038000&cquery=100H_100K_100o_100w_100B&dy_q=1748168444&feature_id=10cf95ef75b4f3e7eac623e4ea0ea691&l=202505251820441DD91332211A56296233","https://www.douyin.com/aweme/v1/play/?video_id=v0200fg10000d0nd0vnog65s0g9k36kg&line=0&file_id=ddab62bed551459687ac7db80fc6a84e&sign=74cabc68c425c171d8592aeb20c266d7&is_play_url=1&source=PackSourceEnum_AWEME_DETAIL"],"width":3840},"quality_type":72`


视频链接：
https://www.douyin.com/aweme/v1/play/?video_id=v0200fg10000d0nd0vnog65s0g9k36kg&line=0&file_id=ddab62bed551459687ac7db80fc6a84e&sign=74cabc68c425c171d8592aeb20c266d7&is_play_url=1&source=PackSourceEnum_AWEME_DETAIL
