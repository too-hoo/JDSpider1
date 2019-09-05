# 模拟浏览器登录JD_WaWaSpider


## 抓取数据并做出分析流程

1、如何分析并找出加载数据的url

- 点开浏览器的F12，使用工具进行抓包，搜索关键字，得到想要的数据包，主要是抓取json数据

2、如何使用requests库的headers解决Referer和User—Agent反扒技术

在请求头中加上参数Referer和User—Agent进行防反扒：
```
headers = {
    'referer': 'https://item.jd.com/1263013576.html',
    'user-agent': 'Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/71.0.3578.98 Safari/537.36'
}
```

3、如何找出分页参数实现批量抓取

找出分页主要就是分析URL，获取多条URL的链接进行对比，特别留意关键字page等等。例如这里就是更改page的值即可：
```
url = 'https://sclub.jd.com/comment/productPageComments.action?callback=fetchJSON_comment98vv6157&productId=1263013576&score=0&sortType=5&page={}&pageSize=10&isShadowSku=0&rid=0&fold=1'.format(page)
```

4、设置一个爬虫间隔时间防止被封ip

在抓取数据的时候不能太频繁，有设置一个时间间隙，不能过多占用本人的服务器的资源：循环抓取100页数据，没抓取一页数据就模拟用户，设置一个随机的时间间隔。
```
for i in range(100):
    spider_comment(i)
    # 模拟用户浏览，设置一个爬虫间隔，防止IP被封
    time.sleep(random.random()*5)
```

5、数据的提取与保存到文件

使用with关键字保存文件，以追加和换行的方式将数据保存到文件：
```
for r_json_comment in r_json_comments:
    # 以追加的方式换行写入每条评论
    with open(COMMENT_FILE_PATH, 'a+') as file:
        file.write(r_json_comment['content']+'\n')
    # 打印评论对象中的评论内容
    print(r_json_comment['content'])
```

6、使用jieba库对数据分词清洗

因为是中文，所以使用中文的jieba分词：
```
def cut_word():
    """
    对数据进行清理
    ：return：分词后的数据
    """
    with open(COMMENT_FILE_PATH) as file:
        comment_txt = file.read()
        wordlist = jieba.cut(comment_txt, cut_all=True)
        wl = " ".join(wordlist)
        # print(wl)
        return wl
```

7、使用wordcloud生成指定的词云

设置词云形状图片，最后生成词云，并将图片保存到本地：
```
def create_word_cloud():
    """
    生成词云
    ：return:
    """
    # 设置词云形状图片
    coloring = np.array(Image.open(WC_MASK_IMG))
    # 设置词云的一些配置，如：字体，背景色，词云形状，大小
    wc = WordCloud(
        background_color="white",
        max_words=2000,
        mask=coloring,
        scale=4,
        max_font_size=50,
        random_state=42,
        font_path=WC_font_PATH)
    # 生成词云
    word_cloud = wc.generate(cut_word())
    # 写词云文件到本地
    word_cloud.to_file("wawa_wc.jpg")
    
    # 在只设置mask的情况下，将会得到一个拥有图片形状的词云
    plt.imshow(wc, interpolation="bilinear")
    plt.axis("off")
    plt.figure()
    plt.show()
```

成果显示：
![](wawa_wc.jpg)



