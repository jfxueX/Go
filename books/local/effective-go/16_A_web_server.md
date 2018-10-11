## A web server

## 一个 Web 服务器

Let's finish with a complete Go program, a web server. This one is actually a kind of web re-server. 
Google provides a service at http://chart.apis.google.com that does automatic formatting of data 
into charts and graphs. It's hard to use interactively, though, because you need to put the data 
into the URL as a query. The program here provides a nicer interface to one form of data: given a 
short piece of text, it calls on the chart server to produce a QR code, a matrix of boxes that 
encode the text. That image can be grabbed with your cell phone's camera and interpreted as, for 
instance, a URL, saving you typing the URL into the phone's tiny keyboard.

让我们以一个完整的 Go 程序作为结束吧，一个 Web 服务器。该程序其实只是个 Web 服务器的重用。 Google 在 
http://chart.apis.google.com 上提供了一个将表单数据自动转换为图表的服务。不过，该服务很难交互， 因为
你需要将数据作为查询放到 URL 中。此程序为一种数据格式提供了更好的的接口： 给定一小段文本，它将调用图
表服务器来生成二维码（QR 码），这是一种编码文本的点格矩阵。 该图像可被你的手机摄像头捕获，并解释为一
个字符串，比如 URL， 这样就免去了你在狭小的手机键盘上键入 URL 的麻烦。

Here's the complete program. An explanation follows.

以下为完整的程序，随后有一段解释。

```go
package main

import (
    "flag"
    "html/template"
    "log"
    "net/http"
)

var addr = flag.String("addr", ":1718", "http service address") // Q=17, R=18

var templ = template.Must(template.New("qr").Parse(templateStr))

func main() {
    flag.Parse()
    http.Handle("/", http.HandlerFunc(QR))
    err := http.ListenAndServe(*addr, nil)
    if err != nil {
        log.Fatal("ListenAndServe:", err)
    }
}

func QR(w http.ResponseWriter, req *http.Request) {
    templ.Execute(w, req.FormValue("s"))
}

const templateStr = `
<html>
<head>
<title>QR Link Generator</title>
</head>
<body>
{{if .}}
<img src="http://chart.apis.google.com/chart?chs=300x300&cht=qr&choe=UTF-8&chl={{.}}" />
<br>
{{.}}
<br>
<br>
{{end}}
<form action="/" name=f method="GET"><input maxLength=1024 size=70
name=s value="" title="Text to QR Encode"><input type=submit
value="Show QR" name=qr>
</form>
</body>
</html>
`
```
The pieces up to main should be easy to follow. The one flag sets a default HTTP port for our 
server. The template variable templ is where the fun happens. It builds an HTML template that will 
be executed by the server to display the page; more about that in a moment.

main 之前的代码应该比较容易理解。我们通过一个标志为服务器设置了默认端口。 模板变量 templ 正是有趣的
地方。它构建的 HTML 模版将会被服务器执行并显示在页面中。 稍后我们将详细讨论。

The main function parses the flags and, using the mechanism we talked about above, binds the 
function QR to the root path for the server. Then http.ListenAndServe is called to start the server; 
it blocks while the server runs.

main 函数解析了参数标志并使用我们讨论过的机制将 QR 函数绑定到服务器的根路径。然后调用 
http.ListenAndServe 启动服务器；它将在服务器运行时处于阻塞状态。

QR just receives the request, which contains form data, and executes the template on the data in the 
form value named s.

QR 仅接受包含表单数据的请求，并为表单值 s 中的数据执行模板。

The template package html/template is powerful; this program just touches on its capabilities. In 
essence, it rewrites a piece of HTML text on the fly by substituting elements derived from data 
items passed to templ.Execute, in this case the form value. Within the template text (templateStr), 
double-brace-delimited pieces denote template actions. The piece from `{{if .}}` to `{{end}} 
`executes only if the value of the current data item, called . (dot), is non-empty. That is, when 
the string is empty, this piece of the template is suppressed.

模板包 html/template 非常强大；该程序只是浅尝辄止。 本质上，它通过在运行时将数据项中提取的元素（在这
里是表单值）传给 templ.Execute 执行因而重写了 HTML 文本。 在模板文本（templateStr）中，双大括号界定
的文本表示模板的动作。 从 `{{if .}}` 到 `{{end}}` 的代码段仅在当前数据项（这里是点 .）的值非空时才会
执行。 也就是说，当字符串为空时，此部分模板段会被忽略。

The two snippets `{{.}}` say to show the data presented to the template—the query string—on the web 
page. The HTML template package automatically provides appropriate escaping so the text is safe to 
display.

其中两段 `{{.}}` 表示要将数据显示在模板中 （即将查询字符串显示在 Web 页面上）。HTML 模板包将自动对文
本进行转义， 因此文本的显示是安全的。

The rest of the template string is just the HTML to show when the page loads. If this is too quick 
an explanation, see the [documentation](https://go-zh.org/pkg/html/template/) for the template 
package for a more thorough discussion.

余下的模板字符串只是页面加载时将要显示的 HTML。如果这段解释你无法理解，请参考 [文档](https://go-
zh.org/pkg/html/template/) 获得更多有关模板包的解释。

And there you have it: a useful web server in a few lines of code plus some data-driven HTML text. 
Go is powerful enough to make a lot happen in a few lines.

你终于如愿以偿了：以几行代码实现的，包含一些数据驱动的HTML文本的Web服务器。 Go语言强大到能让很多事情
以短小精悍的方式解决。
