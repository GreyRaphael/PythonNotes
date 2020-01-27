# Scraper Pool

- [Scraper Pool](#scraper-pool)
  - [Flask-Redis supported proxy pool](#flask-redis-supported-proxy-pool)

## Flask-Redis supported proxy pool

代理池需要定期检查、更新，其中Redis提供队列存储，Flask提供API返回代理.架构图如下
> ![](res/proxy_pool01.png)

其中定时更新器的原理: 从Redis的list的左侧取出proxy，去请求baidu，如果有效添加回Redis的list右侧，无效就抛弃；

[ProxyPool](https://github.com/Python3WebSpider/ProxyPool)

[CookiePool](https://github.com/Germey/CookiesPool)

example: simple flask server

```py
from flask import Flask
app = Flask(__name__)


@app.route("/")
def home():
    return "Hello, Flask!"

@app.route("/index")
def index():
    return "Hello, Index!"

if __name__ == "__main__":
    app.run()
```

example: flask with graph(pyecharts)

```
.
|-- templates
|   `-- pyecharts.html
`-- app.py
```

```html
<!-- pyecharts.html -->
<!DOCTYPE html>
<html>
  <head>
    <meta charset="utf-8" />
    <title>My Graph</title>
    {% for js_name in js_list %}
    <script src="{host}{js_name}"></script>
    {% endfor %}
  </head>

  <body>
    {{myechart|safe}}
    <br />
  </body>
</html>
```

```py
# app.py
from flask import Flask, render_template

import random
from pyecharts import options as opts
from pyecharts.charts import Bar3D
from pyecharts.faker import Faker

app = Flask(__name__)

def bar3d_base() -> Bar3D:
    data = [(i, j, random.randint(0, 12)) for i in range(6) for j in range(24)]
    c = (
        Bar3D()
        .add(
            "",
            [[d[1], d[0], d[2]] for d in data],
            xaxis3d_opts=opts.Axis3DOpts(Faker.clock, type_="category"),
            yaxis3d_opts=opts.Axis3DOpts(Faker.week_en, type_="category"),
            zaxis3d_opts=opts.Axis3DOpts(type_="value"),
        )
        .set_global_opts(
            visualmap_opts=opts.VisualMapOpts(max_=20),
            title_opts=opts.TitleOpts(title="Bar3D-基本示例"),
        )
    )
    return c


@app.route("/index")
def index():
    obj = bar3d_base()
    js_host=obj.js_host
    js_list=obj.js_dependencies.items
    graph_html=obj.render_embed()
    return render_template('pyecharts.html', js_list=js_list, host=js_host, myechart=graph_html)


if __name__ == "__main__":
    app.run()
```