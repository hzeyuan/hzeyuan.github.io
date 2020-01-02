---
layout: post
title: "go爬虫框架-colly实战(四)--知乎回答爬取(一)"
subtitle: 'go爬虫框架-colly实战(四)--知乎回答爬取(二)'
author: "Hzy"
header-style: text
tags:
  - 计算机网络
---

> 原文连接:[Hzy 博客](https://hzeyuan.cn)

## 1.前言

> 好几天没有写啦，这两天发现，每次写爬虫都要自己粘贴复制cookie，感觉好麻烦，colly有个setCookies,之前没明白怎么使用，现在明白啦。

```
siteCokkie :=c.Cookies(URL string)
c.SetCookies(URL string,siteCokkie)
```

>这样子，就能设置某个url访问时的cookie啦，cookies一般是上个请求的cookies，然后我们根据情况来选择是否需要修改cookies。

## 2. 之前知乎上面看到的话题，有什么好看的番剧推荐，我就想到用爬虫爬下来，然后统计出有哪些好看的番剧啦。(好看的番剧都看的差不多啦。有点剧荒..)

#### 问题：[有什么好看的番剧（日本电视动画、网络动画、OVA/OAD连续剧作品）吗？](https://www.zhihu.com/question/319017029/answers/updated)

> 今天我们先把问题下面的全部爬取下来，明天在来清洗数据，进行统计!!!
因为现在已经十二点了..我不想秃头。

### 3. 同样的colly框架，只要简单的请求，写入文件中就完事啦！直接上代码。

>一些注意事项和流程：
* 知乎好像每次请求limt好像限制了为20.
* 所以捏，我的思路，请求一次找到totals，就可以知道有多少个回答啦。
* 然后每次20个，20个的抓，放到文件中就好啦。

```
package main

import (
	"encoding/json"
	"fmt"
	"github.com/PuerkitoBio/goquery"
	"github.com/gocolly/colly"
	"github.com/gocolly/colly/extensions"
	"os"
	"strings"
)

func main(){
	file, error := os.OpenFile("./answer.txt", os.O_RDWR|os.O_CREATE, 0766) //创建文件
	if error != nil {
		fmt.Println(error)
	}
	defer file.Close()
	total := 20 //知乎每次限制返回20个回答
	i:=0 //记录是第几个回答
	c:=colly.NewCollector(func(collector *colly.Collector) {
		extensions.RandomUserAgent(collector)
	})
	c.OnRequest(func(request *colly.Request) {
		fmt.Printf("fetch --->%s\n",request.URL.String())
	})
	c.OnResponse(func(response *colly.Response) {

		var f interface{}
		json.Unmarshal(response.Body,&f) //反序列化
		// 找到改问题下的总回答数量是多少
		paging :=f.(map[string]interface{})["paging"]
		total = int(paging.(map[string]interface{})["totals"].(float64))
		// 找到当前url返回数据中的所有回答。
		data :=f.(map[string]interface{})["data"]
		for k,v :=range data.([]interface{}){
			content :=v.(map[string]interface{})["content"]
			reader :=strings.NewReader(content.(string))
			doc,_:=goquery.NewDocumentFromReader(reader)
			file.Write([]byte(fmt.Sprintf("%d:%s\n",i+k,doc.Find("p").Text())))
		}

	})
	questionID := "319017029"
	for ;i<=total;i+=20{
		//c.Visit()
		url :=fmt.Sprintf("https://www.zhihu.com/api/v4/questions/%s/answers?include=data[*].is_normal,admin_closed_comment,reward_info,is_collapsed,annotation_action,annotation_detail,collapse_reason,is_sticky,collapsed_by,suggest_edit,comment_count,can_comment,content,editable_content,voteup_count,reshipment_settings,comment_permission,created_time,updated_time,review_info,relevant_info,question,excerpt,relationship.is_authorized,is_author,voting,is_thanked,is_nothelp,is_labeled,is_recognized,paid_info,paid_info_content;data[*].mark_infos[*].url;data[*].author.follower_count,badge[*].topics&offset=%d&limit=%d&sort_by=updated",questionID,i,20)
		c.Visit(url)
	}
}
```

## 4. 明天把抓下来的数据，进行一些可视化分析，或者统计，go应该也有这方面的库，明天找找看!! 





