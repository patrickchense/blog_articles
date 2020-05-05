## 爬虫



### 爬美剧更新

[colly](#https://github.com/gocolly/colly)

简单例子:

```
func main() {
	c := colly.NewCollector()

	// Find and visit all links
	c.OnHTML("a[href]", func(e *colly.HTMLElement) {
		e.Request.Visit(e.Attr("href"))
	})

	c.OnRequest(func(r *colly.Request) {
		fmt.Println("Visiting", r.URL)
	})

	c.Visit("http://go-colly.org/")
}
```

搜索网站两个

1. https://91mjw.com/, 搜索api: [https://91mjw.com/?s=%E4%BC%A6%E6%95%A6%E9%BB%91%E5%B8%AE](https://91mjw.com/?s=伦敦黑帮)
2. https://www.kuhuiv.com/filter/oumeiju-----------.html， 感觉很难?一开F12跳转百度？？什么操作?
3. 学习Wireshark, 是不是有帮助？