# 可视化数据采集平台
可视化数据采集平台是一个简单易用的爬虫平台，不需要写代码，只要打开目标网页，选择要获取的元素，保存后执行，就可以创建一个爬虫。目前项目部署在 http://192.168.0.227:9999

![可视化数据采集平台](/images/4.png)

##快速编写一个爬虫
选择“新增任务”，输入名称、url添加采集任务，url为你要爬取的目标网页，如果目标网页需要登录或者传参，可在浏览器登录按f12后找到登录cookie，拷贝到headers和cookie中。
![可视化数据采集平台](/images/5.png)
示例URl：
https://search.51job.com/list/000000,000000,0000,00,9,99,%25E8%25B7%25A8%25E5%25A2%2583,2,1.html?lang=c&stype=&postchannel=0000&workyear=99&cotype=99&degreefrom=99&jobterm=99&companysize=99&providesalary=99&lonlat=0%2C0&radius=-1&ord_field=0&confirmdate=9&fromType=&dibiaoid=0&address=&line=&specialarea=00&from=&welfare=
可输入自己需要爬取的网址url
![可视化数据采集平台](/images/6.png)

打开要爬取的网页后，选择添加选择器
![可视化数据采集平台](/images/7.png)

连续选择要选取的元素可自动获取选择元素，例如连续选择列表中的列可自动选取所有列，选择好元素后点击确定
![可视化数据采集平台](/images/8.jpg)

【多元素】务必勾选，否则只会抓取列表中一条。点击保存选择器
![可视化数据采集平台](/images/9.jpg)

列表title元素获取完后，接着选择分页元素
![可视化数据采集平台](/images/10.jpg)
选择完毕后点击下一步，开始任务
![可视化数据采集平台](/images/11.jpg)
![可视化数据采集平台](/images/12.jpg)

爬虫任务已经开启，可以在“采集任务”中看到爬虫任务
![可视化数据采集平台](/images/13.jpg)
![可视化数据采集平台](/images/14.jpg)

任务爬取结束后可导出数据
![可视化数据采集平台](/images/15.jpg)

## 爬虫平台个人说明
此简易爬虫平台是修改开源代码，思路借鉴与chrome浏览器的插件Web Scraper（有兴趣可以安装一下）。请各位对于此平台提出宝贵意见，或者使用中bug也请与我反馈。
