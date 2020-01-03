---
layout: post
title: "go爬虫框架-colly实战(四)--知乎回答爬取(二)--可视化词云"
subtitle: 'go爬虫框架-colly实战(四)--知乎回答爬取(二)--可视化词云'
author: "Hzy"
header-style: text
tags:
  - 计算机网络
---


> 原文连接:[Hzy 博客](https://hzeyuan.cn)



## 今天试试对数据进行一些简单的处理，然后可视化出来，于是我就想到对出现过的动漫进行一些粗略的统计，然后根据`词语频率`来输出成词云！

## 先看下效果图

![知乎好看的番剧词云](/img/ciyun.png)

## 代码的在我的[GitHub](https://github.com/hzeyuan/learnGO)上，里面放了一些在学习go过程中一些小项目。



### 跟着昨天来，昨天把知乎山的回答抓取了下来，放到了一个文件中。

* 首页要从文件中一行一行的读取(每一行就是一个回答)。
* 读取出来的句子，还得进行一些简单的分割，比如就只提取书名号中的动漫。(ps:当然还可以用其他语言分析的库，想python中的jieba，但我在go中好像没有发现类似的库)，那就只要自己动手写个简单的啦。
* 提取出动漫并且统计后，我们就要进行可视化啦，我在github上发现了`go-echarts`

------
### 2.go-charts简单介绍

安装
```
go get -u github.com/go-echarts/go-echarts
```
文档:[https://go-echarts.github.io/go-echarts/](https://go-echarts.github.io/go-echarts/)

>go-ehcharts 用的是百度开源的echarts图表库，并提供了简洁的api。


-----

### 3.准备的都好啦，下面就是打代码的时间啦

#### 3.1 首先是打开文件，然后读取其中的每一行，然后分割找到动漫名称，然后统计。

```
/*
单词统计
*/
//这个结构体时为了实现sort接口使用的，因为map如果按照value不好排序。
type Pair struct {
	Key string
	Value int
}
type PairList []Pair
func (p PairList) Swap(i, j int)      { p[i], p[j] = p[j], p[i] }
func (p PairList) Len() int           { return len(p) }
func (p PairList) Less(i, j int) bool { return p[j].Value < p[i].Value } // 逆序
type WordCount map[string]interface{}

//遇到以下符号，对句子进行分割

func SplitByMoreStr(r rune) bool{
	splitSymbol := []rune("《》<>")
	for _,v:=range(splitSymbol){
		if r == v{
			return true
		}
	}
	return false


}
// 这里切割读取的行，并简单的统计
func (wc WordCount)SplitAndStatistics(s string){
	dist1 := strings.FieldsFunc(s,SplitByMoreStr)

	for _,v :=range(dist1){
		flag :=0
		v = strings.Replace(v," ","",-1)
		for key :=range wc {
			if strings.Index(v,key)!=-1{ //新的字段中包含了map中曾经出现过的字段，则直接+1
				wc[key]=wc[key].(int)+1
				flag =1
			}
		}
		if flag==0{
			if wc[v]==nil{
				wc[v] =1
			}else{
				wc[v]=wc[v].(int)+1
			}

		}
		//fmt.Println(v)
	}
}
// 读取文件的每一行，并进行统计
func (wc WordCount)ReadFile(f *os.File){
	rd := bufio.NewReader(f)
	for{
		line, err := rd.ReadString('\n') //以'\n'为结束符读入一行
		if err != nil || io.EOF == err {
			break
		}
		wc.SplitAndStatistics(line)//切割并统计
	}
}
//这个函数用来排序，展示结果，但没有用到。
func(wc WordCount)AnalysisResut(){
	//将map[string][int] 转成struct 实现sort接口实现排序功能
	pl :=make(PairList,len(wc))
	i:=0
	for k,v :=range(wc){
		pl[i] = Pair{k,v.(int)}
		i++
	}
	sort.Sort(pl)
	for _,pair :=range(pl){
		fmt.Println(pair.Value,pair.Key)
	}

}
```

#### 3.42 切割完后，我们就要输出成词云才算完事。

> 安装了上面的库，就可以啦。

```
//路由，输出词云
func handler(w http.ResponseWriter, _ *http.Request) {
	nwc := charts.NewWordCloud()

	nwc.SetGlobalOptions(charts.TitleOpts{Title: "知乎问题:"})
	wc :=make(wordCount.WordCount)
	f, err := os.Open(wordCount.Path+"answer.txt")
	if err!=nil{
		panic(err)
	}
	defer f.Close()
	wc.ReadFile(f)
	nwc.Add("wordcloud", wc, charts.WordCloudOpts{SizeRange: []float32{14, 250}})
	nwc.Render(w)
}
// 判断文件是否存在
func Exists(path string) bool {
	_, err := os.Stat(path)    //os.Stat获取文件信息
	if err != nil {
		if os.IsExist(err) {
			return true
		}
		return false
	}
	return true
}

func main(){

	if !Exists(wordCount.Path+"answer.txt"){
		wordCount.QuestionAnswer()
	}
	http.HandleFunc("/", handler)
	http.ListenAndServe(":8081", nil)
}
```


### 总结，还是挺有意思的，下次试试用一些更好，更精确的统计方法，这方面应该是自然语言处理方面的问题，哈哈哈，都是听过，但没玩过...

