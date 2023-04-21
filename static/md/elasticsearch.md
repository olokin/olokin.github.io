### 前端直接操作 elasticsearch 用法
浏览器直接通过js操作elasticsearch。这种方式无需后台服务器了，而且程序业务逻辑都在js里面，代码更新直接替换
原有js和html页面即可，快速便捷，但安全性没有任何保证

#### 一、直接使用 axios 请求
```js
// 获取索引数据
const list
axios.get('/_cat/indices?v').then(res => {
  if (res.data) {
    list = res.data.split('\n').filter(v => v).map(v => v.replace(/\s+/g, ' ').split(' '))
    list.shift()
  }
})

// 根据索引获取日志
axios.post(`/log/filebeat-log-2022-01-01,filebeat-log-2022-01-02/_search?pretty`, getSearchParams()).then(res => {
  if (res.data) {
    const data = res.data.hits.hits.map(v => v._source)
  }
})

function getSearchParams() {
  const must = []

  if (this.logType) {
    must.push({
      match: {
        'Message.MainType': this.logType,
      }
    })
  }

  if (this.desc) {
    must.push({
      match: {
        'Message.Msg': this.desc,
      }
    })
  }

  const obj = {
    query: { bool: { must, } },
    sort: { '@timestamp': { order: 'desc' } },
    from: this.currentPage,
    size: this.currentRows,
  }

  return obj
}
```


#### 二、jquery 写法
```html
  <div id="EsMessage"></div>
```

```js
// 引入插件
<script src="https://cdn.bootcss.com/jquery/3.3.1/jquery.min.js"></script>
<script src="https://cdn.bootcss.com/elasticsearch/14.1.0/elasticsearch.jquery.js"></script>

mounted() {
    //连接的服务器
    let client = new $.es.Client({ hosts: '192.168.251.7:9200', })

    // 获取状态，参数可选，可以只传递一个回调
    client.cluster.health(function (err, resp) {
      if (err) {
        $('#EsMessage').append(err.message + '<br\>')
      } else {
        $('#EsMessage').append('server status: ' + resp.status + '<br\>')
      }
    })

    let data = {
      title: 'test!',
      content: 'It all started when...',
      date: '2018-03-11',
    }

    // 建立索引, 添加数据
    client.index(
      {
        index: 'blog',
        type: 'post',
        id: 1,
        body: data,
      },
      function (err, resp) {
        $('#EsMessage').append('add data result: ' + resp.result + '<br\>')
      }
    )

    // 搜索文档
    const params = {
      index: 'spnews',
      type: 'news',
      body: {
        query: {
          bool: {
            must: [
              {
                term: {
                  title: '足球',
                },
              },
              {
                range: {
                  postdate: {
                    gte: '2016-01-01 10:00:00',
                    format: 'yyyy-MM-dd HH:mm:ss',
                  },
                },
              },
            ],
          },
        },
      },
    }
    client.search({
      index: 'blog',
      size: 50,
      body: {
        query: {
          match: {
            title: 'test',
          },
        },
      },
    }).then(function (resp) {
      $('#EsMessage').append(
        'search data total num : ' + resp.hits.total + '<br\>'
      )
      $('#EsMessage').append(
        'resp.hits.hits[0]._source.title: ' +
          resp.hits.hits[0]._source.title +
          '<br>total number: ' +
          resp.hits.total.value +
          '<br\>'
      )
    })
  },
```
