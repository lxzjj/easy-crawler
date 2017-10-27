# easy-crawler

一个简单的网页爬虫，可以快速抓取特定网页内容


# API Design
```javascript
let crawler = new EasyCrawler({
    max_parallel: 10,
    max_level: Infinity,
    request_timeout: 3000
})

let saveImage = crawler.saveRaw(__dirname)

let fetchHomePage = crawler.createHandler({
    fetch(url, callback) {
        let options = {
            url,
            encoding: null,
            timeout: this.options.timeout
        }

        this.request(options, callback)
    },
    onFetch(response, callback) {
        let html = response.body
        let $ = cheerio.load(html)

        let images = $("image").map(select("src")).get()
        let detailPageUrls = $("a").map(select("href")).get()

        let tasks = [
            ...images.map(saveImage),
            ...detailPageUrls.map(fetchDetailPage),
        ]
        
        callback(null, tasks)
    }
})

let saveArticle = crawler.saveText(__dirname)

let fetchDetailPage = crawler.createHandler((response, callback) => {
    let html = response.body
    let $ = cheerio.load(html)

    let articles = $("article").map((_, el) => $(el).html()).get()
    let tasks = articles.map(saveArticle)

    callback(null, tasks)
})


// start crawler
let entryTask = fetchHomePage("http://www.example.com/homepage/index.html")

entryTask.then((stats) => {
    console.log("done", stats)
}).catch(error => {
    console.error("failed", error)
})

```
