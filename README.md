
#### [How Web Works / vasanthk 英文原版](https://github.com/vasanthk/how-web-works)

# 網際網路是如何運作的

當我們在瀏覽器輸入 google.com 後，在我們不知道的幕後發生了那些事情？

**內容列表**
- [當按下了 google 字首的 "g" 按鍵時](#當按下了google字首的g按鍵時)
- [當你按了 "Enter" 鍵時](#當你按了enter鍵時)
- [解析 URL](#解析url)
- [檢視 HTTP 嚴格傳輸安全模式](#檢視http嚴格傳輸安全模式)
- [DNS 查找](#DNS查找)
- [Opening of a socket + TLS handshake](#opening-of-a-socket--tls-handshake)
- ["HTTP" 超文本傳輸協定](#http超文本傳輸協定)
- [對於 HTTP 伺服器的請求處理](#對於http伺服器的請求處理)
- [伺服器回應](#伺服器回應)
- [瀏覽器的背後都在做甚麼](#瀏覽器的背後都在做甚麼)
- [瀏覽器高階層級結構](#瀏覽器高階層級結構)
- [畫面渲染引擎](#畫面渲染引擎)
- [主要流程](#主要流程)
- [Parsing Basics](#parsing-basics)
- [文件物件模型樹](#文件物件模型樹)
- [渲染樹](#渲染樹)
- [渲染樹與文件物件模型樹的關係](#渲染樹與文件物件模型樹的關係)
- [CSS 解析](#css解析)
- [畫面佈局](#畫面佈局)
- [描繪](#描繪)
- [Trivia](#trivia)


## 當按下了google字首的g按鍵時

當你點選瀏覽器上方的 URL 欄位，並按下鍵盤上的 g 按鍵時，瀏覽器會接收到這個觸發事件，接著這個強大的自動機器就會以迅雷不及掩耳的速度運作起來。
你的瀏覽器會以它本身的演算法及依照你所選擇模式來決定瀏覽器整體的運作方式；例如：你是否選擇使用 "隱私模式/無痕模式" 將會決定你在 URL 輸入網址字串時，URL 欄位下方會不會出現預測的建議網址字串。
這些 URL 網址預測的演算法，大部分是優先考慮你的 "搜尋歷史" 及 "我的最愛書籤" ; 
這時你決定要在 URL 網址欄內輸入 google.com，在你開始輸入第一個文字時，可能會出現很多不相關的建議網址出現在 URL 欄列下方，但是每當你多輸入一個字母時，瀏覽器的系統背景裡有很多的程式碼會同步的運作起來，這些背景程式會一步一步的修正要顯示給你的建議內容，最後可能在你還沒完整輸入 google.com 整段字串時，它就已經知道你要輸入 google.com 並且顯示給你看。


## 當你按了Enter鍵時
我們來檢視一個瞬間的動作點，當我們來按下鍵盤上的 enter 鍵的那瞬間，
連接到這個按鍵的電子迴路就會導通( 迴路不論是直接連通或經由電容導通 )，微小的電流就會在鍵盤上的邏輯電路中流動，經由電流的流動可以知道鍵盤上每個按鍵的開關狀態，降低開關在快速間歇性閉合的情況下所造成的電子雜訊，接著再把我們我按下的按鍵的電子訊號轉換成按鍵的整數編號，在這個例子中，enter 鍵的編號是13。鍵盤控制器再將鍵盤編號進行編碼並傳送給電腦核心。現在世界上大部分的電腦硬體，主要都是使用 USB 及藍芽傳輸來讓鍵盤這個輸入器跟電腦的核心連接。


在鍵盤的例子中：

* 按鍵編號是被儲存在端點暫存器的內部記憶電路中。
* 主機上的 USB 控制器每 10 ms 左右會與鍵盤的端點進行通訊，所以可以知道已經被儲存在端點上的按鍵編號。
* 在 USB 2.0 的序列介面引擎上，這些數值可以最大每秒 1.5 MB 的速度來傳送
* 這些序列訊號是在電腦主機上的 USB 控制器被解碼，被經由 HID 通用鍵盤設備的驅動程式來解釋意義
* 最後這些數值都會被傳送到作業系統的硬體抽象層級來運用


在觸控螢幕的例子中:

* 當使用者的手指放在先進的觸控螢幕上時，會有非常小的電流轉移到使用者的手指上，這完整的迴路是透過導電層的靜電場，在螢幕的觸碰點產生一個電壓降，螢幕控制器就會回傳訊號來表示被點擊的座標
* 接著，行動裝置的作業系統就會通知在螢幕前景運作的 App 的圖形使用者介面元素以及應該要被觸發的點擊事件來進行運作 (這裡指的是虛擬的鍵盤按鍵)。
* 這個虛擬鍵盤這時突然打岔的回傳一個 "按鍵被使用了" 的通知給作業系統
* 這個打岔的回傳訊號通知了正在前景使用中的 App 一個 "按鍵被使用了" 事件


## 解析URL

現在在瀏覽器上的 URL 網址列中有以下的相關資訊

* HTTP 協定:全名稱為 "超文本傳輸協定"
* 所取得的首頁檔案 (或稱做資源) 會有 "/" 斜線符號來分隔

當輸入的 URL 文字不是有效的協定或網域名稱時，瀏覽器就會把原本輸入在的 URL 的文字傳送到瀏覽器預設的搜尋引擎去進行搜尋。


## 檢視HTTP嚴格傳輸安全模式

* 瀏覽器在對伺服器發出請求之前，會去檢視一個預先準備好的列表叫做 "HTTP 嚴格傳輸安全" 列表，
這個表單上紀載的是哪些網站曾經要求過必須只能使用 HTTPS 來進行連結 (HTTP secure ，一個用 HTTP 傳輸，並使用 SSL/TLS 來加密傳輸的封包) 

* 如果你要找的網站網址在這個列表上，它就會使用 HTTPS 來傳輸你的請求給伺服器。如果不在列表上的話，就會使用預設的 HTTP 來傳輸請求。

註記： 
如果網站本身有提供 HTTPS，那就算網站網址沒有在瀏覽器的 HTTP 嚴格傳輸安全列表內，那仍然可以使用 HTTPS。
當使用者第一次發出 HTTP 的請求，伺服器就會回傳給使用者，要求使用者只能使用 HTTPS 來傳送請求。
然而，這種單次的 HTTP 發送請求，有潛在會洩漏使用者弱點的風險而造成 [降級攻擊](http://www.yourdictionary.com/downgrade-attack)，而 HTTP 嚴格傳輸安全列表就是為了這種資安考量，而設置在現代的瀏覽器內。


## DNS查找

瀏覽器會試著把 IP 位址對應到曾經進入過的網域。DNS(網域域名伺服器)在進行查找的過程中，會進行以下的流程：

* **瀏覽器快取：** 瀏覽器在某些時候會儲存 DNS 的紀錄。有趣的是，作業系統並不會告訴瀏覽器，每個 DNS 中的每筆資料會存留多久，所以瀏覽器會在固定的時間間隔來儲存這些域名資料(這個間隔時間大約 2 到 30 分鐘之間)。

* **作業系統快取:** 如果瀏覽器所儲存的快取資料並不是我們想要的，那瀏覽器就會對系統發出一個功能請求 (在 windows 系統中是使用 gethostbyname 函式)，作業系統有自己本身的快取。

* **路由器快取:** 這些域名查找的請求也會在你的路由器上執行，在大多數的情況下，路由器本身也會有類似 DNS 的快取域名的功能。

* **網路服務供應商 DNS 快取:** 照常理說，在你當地的網路服務供應商都會提供伺服器來做網域域名查找的服務。

* **遞迴搜尋:** 網路供應商的 DNS 進行著遞迴方式的搜尋，從根域名伺服器、.com 的第一級域名伺服器、一直找到 google 的域名伺服器。
一般來說，DNS 會儲存 .com 的頂級域名伺服器的網址快取，所以實際上並不一定要使用到根域名伺服器。


有關於遞迴方式的 DNS 搜尋示意圖

<p align="center">
  <img src="http://igoro.com/wordpress/wp-content/uploads/2010/02/500pxAn_example_of_theoretical_DNS_recursion_svg.png" alt="Recursive DNS search"/>
</p>

DNS 令人擔心的一件事，就是當如果要搜尋的是一個完整的網域名，像是 wikipedia.org 或是 facebook.com 時，似乎是要把域名對應到一個 IP 位址上。但還好，現在已經有很多方式可以減輕這個瓶頸所帶來的負擔:

* **轉輪循環式 DNS**這是一個 DNS 用來回覆多重 IP 的解決方式。例如: facebook.com 這個域名實際上會對應到四個 IP 位址，那經由這種轉輪循環式的 DNS 的服務，就會給查找 facebook.com 的使用者輪流回覆這四個 IP。

* **負載平衡器** 這是一種硬體，可以設定當收到某些特定 IP 時，就轉發請求到其他的伺服器，流量大的網站一般來說都會使用性能很好、很昂貴的負載平衡器。

* **與地理位置相關的 DNS** 依照使用者所在的地理位置，來回覆該地理位置所被設定對應被查詢網域的 IP 位址，如此可以增加各個服器的擴展性，這對於管理不同的伺服器上的一些靜態內容有好處，也不用去更新共享的狀態。


* **任播** 這是一種路由技術，以單一 IP 位址可以任意連接到多個伺服器，但每次只會有單一目標伺服器會收到請求。較不理想的是，任播較不適合使用 TCP。


大多數的 DNS 伺服器使用任播技術是為了在域名查找的時候，可以做到高可用性及低延遲性。
使用任播服務(DNS 在這邊是很好的例子)的使用者會永遠連接到最近(但主要還是看個別路由的演算法而定)的 DNS 伺服器。這可以減低網路延遲，也可以平均分散每個伺服器的負載(這假定你的使用者是很平均分布在你的網路環境中)

## Opening of a socket + TLS handshake

* Once the browser receives the IP address of the destination server, it takes that and the given port number from the URL (the HTTP protocol defaults to port 80, and HTTPS to port 443), and makes a call to the system library function named socket and requests a [TCP](http://www.webopedia.com/TERM/T/TCP.html) [socket](http://www.webopedia.com/TERM/S/socket.html) stream.
* The client computer sends a ClientHello message to the server with its TLS version, list of cipher algorithms and compression methods available.
* The server replies with a ServerHello message to the client with the TLS version, selected cipher, selected compression methods and the server's public certificate signed by a CA (Certificate Authority). The certificate contains a public key that will be used by the client to encrypt the rest of the handshake until a symmetric key can be agreed upon.
* The client verifies the server digital certificate against its list of trusted CAs. If trust can be established based on the CA, the client generates a string of pseudo-random bytes and encrypts this with the server's public key. These random bytes can be used to determine the symmetric key.
* The server decrypts the random bytes using its private key and uses these bytes to generate its own copy of the symmetric master key.
* The client sends a Finished message to the server, encrypting a hash of the transmission up to this point with the symmetric key.
* The server generates its own hash, and then decrypts the client-sent hash to verify that it matches. If it does, it sends its own Finished message to the client, also encrypted with the symmetric key.
* From now on the TLS session transmits the application (HTTP) data encrypted with the agreed symmetric key.

## HTTP protocol

You can be pretty sure that dynamic sites such as Facebook/Gmail will not be served from the browser cache because dynamic pages expire either very quickly or immediately (expiry date set to past).

If the web browser used was written by Google, instead of sending an HTTP request to retrieve the page, it will send a request to try and negotiate with the server an "upgrade" from HTTP to the SPDY protocol. Note that SPDY is being deprecated in favor of HTTP/2 in latest versions of Chrome.

```txt
GET http://www.google.com/ HTTP/1.1
Accept: application/x-ms-application, image/jpeg, application/xaml+xml, [...]
User-Agent: Mozilla/4.0 (compatible; MSIE 8.0; Windows NT 6.1; WOW64; [...]
Accept-Encoding: gzip, deflate
Connection: Keep-Alive
Host: google.com
Cookie: datr=1265876274-[...]; locale=en_US; lsd=WW[...]; c_user=2101[...]
```

The GET request names the URL to fetch: “http://www.google.com/”. The browser identifies itself (User-Agent header), and states what types of responses it will accept (Accept and Accept-Encoding headers). The Connection header asks the server to keep the TCP connection open for further requests.

The request also contains the cookies that the browser has for this domain. As you probably already know, cookies are key-value pairs that track the state of a web site in between different page requests. And so the cookies store the name of the logged-in user, a secret number that was assigned to the user by the server, some of user’s settings, etc. The cookies will be stored in a text file on the client, and sent to the server with every request.

HTTP/1.1 defines the "close" connection option for the sender to signal that the connection will be closed after completion of the response. For example, Connection: close.

After sending the request and headers, the web browser sends a single blank newline to the server indicating that the content of the request is done. The server responds with a response code denoting the status of the request and responds with a response of the form: **200 OK [response headers]**

Followed by a single newline, and then sends a payload of the HTML content of www.google.com. The server may then either close the connection, or if headers sent by the client requested it, keep the connection open to be reused for further requests.

If the HTTP headers sent by the web browser included sufficient information for the web server to determine if the version of the file cached by the web browser has been unmodified since the last retrieval (ie. if the web browser included an ETag header), it may instead respond with a request of the form: **304 Not Modified [response headers]** and no payload, and the web browser instead retrieves the HTML from its cache.

After parsing the HTML, the web browser (and server) repeats this process for every resource (image, CSS, favicon.ico, etc) referenced by the HTML page, except instead of GET / HTTP/1.1 the request will be **GET /$(URL relative to www.google.com) HTTP/1.1.**

If the HTML referenced a resource on a different domain than www.google.com, the web browser goes back to the steps involved in resolving the other domain, and follows all steps up to this point for that domain. The Host header in the request will be set to the appropriate server name instead of google.com.

**Gotcha:** 
* The trailing slash in the URL “http://facebook.com/” is important. In this case, the browser can safely add the slash. For URLs of the form http://example.com/folderOrFile, the browser cannot automatically add a slash, because it is not clear whether folderOrFile is a folder or a file. In such cases, the browser will visit the URL without the slash, and the server will respond with a redirect, resulting in an unnecessary roundtrip.
* The server might respond with a 301 Moved Permanently response to tell the browser to go to “http://www.google.com/” instead of “http://google.com/”. There are interesting reasons why the server insists on the redirect instead of immediately responding with the web page that the user wants to see.
One reason has to do with search engine rankings. See, if there are two URLs for the same page, say http://www.vasanth.com/ and http://vasanth.com/, search engine may consider them to be two different sites, each with fewer incoming links and thus a lower ranking. Search engines understand permanent redirects (301), and will combine the incoming links from both sources into a single ranking. 
Also, multiple URLs for the same content are not cache-friendly. When a piece of content has multiple names, it will potentially appear multiple times in caches.

**Note:**
HTTP response starts with the returned status code from the server. Following is a very brief summary of what a status code denotes:        
  * 1xx indicates an informational message only
  * 2xx indicates success of some kind
  * 3xx redirects the client to another URL
  * 4xx indicates an error on the client's part
  * 5xx indicates an error on the server's part
 
## HTTP Server Request Handle

The HTTPD (HTTP Daemon) server is the one handling the requests/responses on the server side. The most common HTTPD servers are Apache or nginx for Linux and IIS for Windows.

* The HTTPD (HTTP Daemon) receives the request.

* The server breaks down the request to the following parameters:
    * HTTP Request Method (either GET, POST, HEAD, PUT and DELETE). In the case of a URL entered directly into the address bar, this will be GET.
    * Domain, in this case - google.com.
    * Requested path/page, in this case - / (as no specific path/page was requested, / is the default path).
    * The server verifies that there is a Virtual Host configured on the server that corresponds with google.com.

* The server verifies that google.com can accept GET requests.

* The server verifies that the client is allowed to use this method (by IP, authentication, etc.).

* If the server has a rewrite module installed (like mod_rewrite for Apache or URL Rewrite for IIS), it tries to match the request against one of the configured rules. If a matching rule is found, the server uses that rule to rewrite the request.

* The server goes to pull the content that corresponds with the request, in our case it will fall back to the index file, as "/" is the main file (some cases can override this, but this is the most common method).

* The server parses the file according to the request handler. A request handler is a program (in ASP.NET, PHP, Ruby, …) that reads the request and generates the HTML for the response. If Google is running on PHP, the server uses PHP to interpret the index file, and streams the output to the client.

Note: One interesting difficulty that every dynamic website faces is how to store data. Smaller sites will often have a single SQL database to store their data, but sites that store a large amount of data and/or have many visitors have to find a way to split the database across multiple machines. Solutions include sharding (splitting up a table across multiple databases based on the primary key), replication, and usage of simplified databases with weakened consistency semantics.

## Server Response

Here is the response that the server generated and sent back:

```txt
HTTP/1.1 200 OK
Cache-Control: private, no-store, no-cache, must-revalidate, post-check=0,
    pre-check=0
Expires: Sat, 01 Jan 2000 00:00:00 GMT
P3P: CP="DSP LAW"
Pragma: no-cache
Content-Encoding: gzip
Content-Type: text/html; charset=utf-8
X-Cnection: close
Transfer-Encoding: chunked
Date: Fri, 12 Feb 2010 09:05:55 GMT

2b3
��������T�n�@����[...]
```

The entire response is 36 kB, the bulk of them in the byte blob at the end that I trimmed.

The **Content-Encoding** header tells the browser that the response body is compressed using the gzip algorithm. After decompressing the blob, you’ll see the HTML you’d expect:

```html
<!DOCTYPE html PUBLIC "-//W3C//DTD XHTML 1.0 Strict//EN"   
      "http://www.w3.org/TR/xhtml1/DTD/xhtml1-strict.dtd">
<html xmlns="http://www.w3.org/1999/xhtml" xml:lang="en" 
      lang="en" id="google" class=" no_js">
<head>
<meta http-equiv="Content-type" content="text/html; charset=utf-8" />
<meta http-equiv="Content-language" content="en" />
...
```

Notice the header that sets Content-Type to text/html. The header instructs the browser to render the response content as HTML, instead of say downloading it as a file. The browser will use the header to decide how to interpret the response, but will consider other factors as well, such as the extension of the URL.

## Behind the scenes of the Browser
   
Once the server supplies the resources (HTML, CSS, JS, images, etc.) to the browser it undergoes the below process:
* Parsing - HTML, CSS, JS
* Rendering - Construct DOM Tree → Render Tree → Layout of Render Tree → Painting the render tree

## 瀏覽器高階層級結構

1. **User Interface:** Includes the address bar, back/forward button, bookmarking menu, etc. Every part of the browser display except the window where you see the requested page.

2. **Browser Engine:** [Marshals](http://stackoverflow.com/a/5600887/1672655) actions between the UI and the rendering engine.

3. **Rendering Engine:** Responsible for displaying requested content. For eg. the rendering engine parses HTML and CSS, and displays the parsed content on the screen.

4. **Networking:** For network calls such as HTTP requests, using different implementations for different platforms (behind a platform-independent interface).

5. **UI Backend:** Used for drawing basic widgets like combo boxes and windows. This backend exposes a generic interface that is not platform specific. Underneath it uses operating system user interface methods.

6. **JavaScript Engine:** Interpreter used to parse and execute JavaScript code.

7. **Data Storage:** This is a persistence layer. The browser may need to save data locally, such as cookies. Browsers also support storage mechanisms such as [localStorage](https://developer.mozilla.org/en-US/docs/Web/API/Window/localStorage), [IndexedDB](https://developer.mozilla.org/en-US/docs/Web/API/IndexedDB_API/Using_IndexedDB) and [FileSystem](https://developer.chrome.com/apps/fileSystem).

<p align="center">
  <img src="http://www.html5rocks.com/en/tutorials/internals/howbrowserswork/layers.png" alt="Browser Components"/>
</p>

Let’s start, with the simplest possible case: a plain HTML page with some text and a single image. What does the browser need to do to process this simple page?

1. **Conversion:** the browser reads the raw bytes of the HTML off the disk or network and translates them to individual characters based on specified encoding of the file (e.g. UTF-8).

2. **Tokenizing:** the browser converts strings of characters into distinct tokens specified by the W3C HTML5 standard - e.g. “<html>”, “<body>” and other strings within the “angle brackets”. Each token has a special meaning and a set of rules.

3. **Lexing:** the emitted tokens are converted into “objects” which define their properties and rules.

4. **DOM construction:** Finally, because the HTML markup defines relationships between different tags (some tags are contained within tags) the created objects are linked in a tree data structure that also captures the parent-child relationships defined in the original markup: HTML object is a parent of the body object, the body is a parent of the paragraph object, and so on.

<p align="center">
  <img src="https://developers.google.com/web/fundamentals/performance/critical-rendering-path/images/full-process.png" alt="DOM Construction Process"/>
</p>

The final output of this entire process is the Document Object Model, or the “DOM” of our simple page, which the browser uses for all further processing of the page.

Every time the browser has to process HTML markup it has to step through all of the steps above: convert bytes to characters, identify tokens, convert tokens to nodes, and build the DOM tree. This entire process can take some time, especially if we have a large amount of HTML to process.

<p align="center">
  <img src="https://developers.google.com/web/fundamentals/performance/critical-rendering-path/images/dom-timeline.png" alt="Tracing DOM construction in DevTools"/>
</p>

If you open up Chrome DevTools and record a timeline while the page is loaded, you can see the actual time taken to perform this step — in the example above, it took us ~5ms to convert a chunk of HTML bytes into a DOM tree. Of course, if the page was larger, as most pages are, this process might take significantly longer. You will see in our future sections on creating smooth animations that this can easily become your bottleneck if the browser has to process large amounts of HTML.

## Rendering Engine

A rendering engine is a software component that takes marked up content (such as HTML, XML, image files, etc.) and formatting information (such as CSS, XSL, etc.) and displays the formatted content on the screen.

|Browser           |Engine                       |
|----------------- |:---------------------------:|
|Chrome            | Blink (a fork of WebKit)    |
|Firefox           | Gecko                       |
|Safari            | Webkit                      |
|Opera             | Blink (Presto if < v15)     |
|Internet Explorer | Trident                     |
|Edge              | EdgeHTML                    |  

WebKit is an open source rendering engine which started as an engine for the Linux platform and was modified by Apple to support Mac and Windows.

The rendering engine is single threaded. Almost everything, except network operations, happens in a single thread. In Firefox and Safari this is the main thread of the browser. In Chrome it's the tab process main thread. 
Network operations can be performed by several parallel threads. The number of parallel connections is limited (usually 2–6 connections).

The browser main thread is an event loop. It's an infinite loop that keeps the process alive. It waits for events (like layout and paint events) and processes them.

Note: Browsers such as Chrome run multiple instances of the rendering engine: one for each tab. Each tab runs in a separate process.

## The Main flow

The rendering engine will start getting the contents of the requested document from the networking layer. This is usually done in 8KB chunks.

After that the basic flow of the rendering engine is:

<p align="center">
  <img src="http://www.html5rocks.com/en/tutorials/internals/howbrowserswork/flow.png" alt="Rendering engine basic flow"/>
</p>

The rendering engine will start parsing the HTML document and convert elements to [DOM](http://domenlightenment.com/) nodes in a tree called the **"content tree"**. 

The engine will parse the style data, both in external CSS files and in style elements. Styling information together with visual instructions in the HTML will be used to create another tree: the **render tree**.
The render tree contains rectangles with visual attributes like color and dimensions. The rectangles are in the right order to be displayed on the screen.

After the construction of the render tree it goes through a **"layout"** process. This means giving each node the exact coordinates where it should appear on the screen.

The next stage is **painting**-the render tree will be traversed and each node will be painted using the UI backend layer.

It's important to understand that this is a gradual process. For better user experience, the rendering engine will try to display contents on the screen as soon as possible. It will not wait until all HTML is parsed before starting to build and layout the render tree. Parts of the content will be parsed and displayed, while the process continues with the rest of the contents that keeps coming from the network.

Given below is Webkit's flow:

<p align="center">
  <img src="http://www.html5rocks.com/en/tutorials/internals/howbrowserswork/webkitflow.png" alt="Webkit main flow"/>
</p>

## Parsing Basics

**Parsing:** Translating the document to a structure the code can use. The result of parsing is usually a tree of nodes that represent the structure of the document.
 
**Grammar:** Parsing is based on the syntax rules the document obeys: the language or format it was written in. Every format you can parse must have deterministic grammar consisting of vocabulary and syntax rules. It is called a **context free grammar**.  

Parsing can be separated into two sub processes: lexical analysis and syntax analysis.
              
**Lexical analysis:** The process of breaking the input into tokens. Tokens are the language vocabulary: the collection of valid building blocks.

**Syntax analysis:** The applying of the language syntax rules.

Parsers usually divide the work between two components: the lexer (sometimes called tokenizer) that is responsible for breaking the input into valid tokens, and the parser that is responsible for constructing the parse tree by analyzing the document structure according to the language syntax rules. The lexer knows how to strip irrelevant characters like white spaces and line breaks.

<p align="center">
  <img src="http://www.html5rocks.com/en/tutorials/internals/howbrowserswork/image011.png" alt="Source document to parse tree"/>
</p>

The parsing process is iterative. The parser will usually ask the lexer for a new token and try to match the token with one of the syntax rules. If a rule is matched, a node corresponding to the token will be added to the parse tree and the parser will ask for another token.

If no rule matches, the parser will store the token internally, and keep asking for tokens until a rule matching all the internally stored tokens is found. If no rule is found then the parser will raise an exception. This means the document was not valid and contained syntax errors.

The job of the HTML parser is to parse the HTML markup into a parse tree. HTML definition is in a DTD (Document Type Definition) format. This format is used to define languages of the SGML family. The format contains definitions for all allowed elements, their attributes and hierarchy. As we saw earlier, the HTML DTD doesn't form a context free grammar.

HTML parsing algorithm consists of two stages: tokenization and tree construction.

**Tokenization** is the lexical analysis, parsing the input into tokens. Among HTML tokens are start tags, end tags, attribute names and attribute values. The tokenizer recognizes the token, gives it to the tree constructor, and consumes the next character for recognizing the next token, and so on until the end of the input.

<p align="center">
  <img src="http://www.html5rocks.com/en/tutorials/internals/howbrowserswork/image017.png" alt="HTML parsing flow"/>
</p>

## DOM Tree

The output tree (the "parse tree") is a tree of DOM element and attribute nodes. DOM is short for Document Object Model. It is the object presentation of the HTML document and the interface of HTML elements to the outside world like JavaScript. The root of the tree is the "Document" object.

The DOM has an almost one-to-one relation to the markup. For example:

```html
<html>
  <body>
    <p>
      Hello World
    </p>
    <div> <img src="example.png"/></div>
  </body>
</html>
```

This markup would be translated to the following DOM tree:

<p align="center">
  <img src="http://www.html5rocks.com/en/tutorials/internals/howbrowserswork/image015.png" alt="DOM Tree"/>
</p>

### Why is the DOM slow?

The short answer is that the DOM is not slow. Adding & removing a DOM node is a few pointer swaps, not much more than setting a property on the JS object.

However, layout is slow. When you touch the DOM in any way, you set a dirty bit on the whole tree that tells the browser it needs to figure out where everything goes again. When JS hands control back to the browser, it invokes its layout algorithm (or more technically, it invokes its CSS recalc algorithm, then layout, then repaint, then re-compositing) to redraw the screen. The layout algorithm is quite complex - read the CSS spec to understand some of the rules - and that means it often has to make non-local decisions.

Worse, layout is triggered synchronously by accessing certain properties. Among those are getComputedStyleValue(), getBoundingClientWidth(), .offsetWidth, .offsetHeight, etc, which makes them stupidly easy to run into. Full list is [here](https://gist.github.com/paulirish/5d52fb081b3570c81e3a).
Because of this, a lot of Angular and JQuery code is stupidly slow. One layout will blow your entire frame budget on a mobile device. When I measured Google Instant c. 2013, it caused 13 layouts in one query, and locked up the screen for nearly 2 seconds on a mobile device. (It's since been sped up.)

React doesn't help speed up layout - if you want butter-smooth animations on a mobile web browser, you need to resort to other techniques like limiting everything you do in a frame to operations that can be performed on the GPU. But what it does do is ensure that there is at most one layout performed each time you update the state of the page. That's often quite an improvement on the status quo.

## Render Tree

While the DOM tree is being constructed, the browser constructs another tree, the render tree. This tree is of visual elements in the order in which they will be displayed. It is the visual representation of the document. The purpose of this tree is to enable painting the contents in their correct order.

A renderer knows how to lay out and paint itself and its children. Each renderer represents a rectangular area usually corresponding to a node's CSS box.

## Render tree's relation to the DOM tree

The renderers correspond to DOM elements, but the relation is not one to one. Non-visual DOM elements will not be inserted in the render tree. An example is the "head" element. Also elements whose display value was assigned to "none" will not appear in the tree (whereas elements with "hidden" visibility will appear in the tree).

There are DOM elements which correspond to several visual objects. These are usually elements with complex structure that cannot be described by a single rectangle. For example, the "select" element has three renderers: one for the display area, one for the drop down list box and one for the button. Also when text is broken into multiple lines because the width is not sufficient for one line, the new lines will be added as extra renderers.
 
Some render objects correspond to a DOM node but not in the same place in the tree. Floats and absolutely positioned elements are out of flow, placed in a different part of the tree, and mapped to the real frame. A placeholder frame is where they should have been.

<p align="center">
  <img src="http://www.html5rocks.com/en/tutorials/internals/howbrowserswork/image025.png" alt="The render tree and the corresponding DOM tree"/>
</p>
 
In WebKit the process of resolving the style and creating a renderer is called "attachment". Every DOM node has an "attach" method. Attachment is synchronous, node insertion to the DOM tree calls the new node "attach" method.

Building the render tree requires calculating the visual properties of each render object. This is done by calculating the style properties of each element. The style includes style sheets of various origins, inline style elements and visual properties in the HTML (like the "bgcolor" property).The later is translated to matching CSS style properties.

## CSS Parsing

CSS Selectors are matched by browser engines from right to left. Keep in mind that when a browser is doing selector matching it has one element (the one it's trying to determine style for) and all your rules and their selectors and it needs to find which rules match the element. This is different from the usual jQuery thing, say, where you only have one selector and you need to find all the elements that match that selector.

A selector's specificity is calculated as follows:

* Count 1 if the declaration it is from is a 'style' attribute rather than a rule with a selector, 0 otherwise (= a)
* Count the number of ID selectors in the selector (= b)
* Count the number of class selectors, attributes selectors, and pseudo-classes in the selector (= c)
* Count the number of element names and pseudo-elements in the selector (= d)
* Ignore the universal selector

Concatenating the three numbers a-b-c-d (in a number system with a large base) gives the specificity. The number base you need to use is defined by the highest count you have in one of a, b, c and d.
 
Examples:

``` txt
*               /* a=0 b=0 c=0 -> specificity =   0 */
LI              /* a=0 b=0 c=1 -> specificity =   1 */
UL LI           /* a=0 b=0 c=2 -> specificity =   2 */
UL OL+LI        /* a=0 b=0 c=3 -> specificity =   3 */
H1 + *[REL=up]  /* a=0 b=1 c=1 -> specificity =  11 */
UL OL LI.red    /* a=0 b=1 c=3 -> specificity =  13 */
LI.red.level    /* a=0 b=2 c=1 -> specificity =  21 */
#x34y           /* a=1 b=0 c=0 -> specificity = 100 */
#s12:not(FOO)   /* a=1 b=0 c=1 -> specificity = 101 */
``` 

Why does the CSSOM have a tree structure? When computing the final set of styles for any object on the page, the browser starts with the most general rule applicable to that node (e.g. if it is a child of body element, then all body styles apply) and then recursively refines the computed styles by applying more specific rules - i.e. the rules “cascade down”. 
 
WebKit uses a flag that marks if all top level style sheets (including @imports) have been loaded. If the style is not fully loaded when attaching, place holders are used and it is marked in the document, and they will be recalculated once the style sheets were loaded. 

## Layout

When the renderer is created and added to the tree, it does not have a position and size. Calculating these values is called layout or reflow.

HTML uses a flow based layout model, meaning that most of the time it is possible to compute the geometry in a single pass. Elements later 'in the flow' typically do not affect the geometry of elements that are earlier 'in the flow', so layout can proceed left-to-right, top-to-bottom through the document. The coordinate system is relative to the root frame. Top and left coordinates are used.

Layout is a recursive process. It begins at the root renderer, which corresponds to the <html> element of the HTML document. Layout continues recursively through some or all of the frame hierarchy, computing geometric information for each renderer that requires it.

The position of the root renderer is 0,0 and its dimensions are the viewport–the visible part of the browser window. All renderers have a "layout" or "reflow" method, each renderer invokes the layout method of its children that need layout. 

In order not to do a full layout for every small change, browsers use a "dirty bit" system. A renderer that is changed or added marks itself and its children as "dirty": needing layout. There are two flags: "dirty", and "children are dirty" which means that although the renderer itself may be OK, it has at least one child that needs a layout.

The layout usually has the following pattern:

- Parent renderer determines its own width.
- Parent goes over children and:
    - Place the child renderer (sets its x and y).
    - Calls child layout if needed–they are dirty or we are in a global layout, or for some other reason–which calculates the child's height.
- Parent uses children's accumulative heights and the heights of margins and padding to set its own height–this will be used by the parent renderer's parent.
- Sets its dirty bit to false.

Also note, layout thrashing is where a web browser has to reflow or repaint a web page many times before the page is ‘loaded’. In the days before JavaScript’s prevalence, websites were typically reflowed and painted just once, but these days it is increasingly common for JavaScript to run on page load which can cause modifications to the DOM and therefore extra reflows or repaints. Depending on the number of reflows and the complexity of the web page, there is potential to cause significant delay when loading the page, especially on lower powered devices such as mobile phones or tablets.

## Painting

In the painting stage, the render tree is traversed and the renderer's "paint()" method is called to display content on the screen. Painting uses the UI infrastructure component.

Like layout, painting can also be global–the entire tree is painted–or incremental. In incremental painting, some of the renderers change in a way that does not affect the entire tree. The changed renderer invalidates its rectangle on the screen. This causes the OS to see it as a "dirty region" and generate a "paint" event. The OS does it cleverly and coalesces several regions into one. 

Before repainting, WebKit saves the old rectangle as a bitmap. It then paints only the delta between the new and old rectangles. The browsers try to do the minimal possible actions in response to a change. So changes to an elements color will cause only repaint of the element. Changes to the element position will cause layout and repaint of the element, its children and possibly siblings. Adding a DOM node will cause layout and repaint of the node. Major changes, like increasing font size of the "html" element, will cause invalidation of caches, relayout and repaint of the entire tree.

There are three different positioning schemes:

* **Normal:** the object is positioned according to its place in the document. This means its place in the render tree is like its place in the DOM tree and laid out according to its box type and dimensions
* **Float:** the object is first laid out like normal flow, then moved as far left or right as possible
* **Absolute:** the object is put in the render tree in a different place than in the DOM tree

The positioning scheme is set by the "position" property and the "float" attribute.

- static and relative cause a normal flow
- absolute and fixed cause absolute positioning

In static positioning no position is defined and the default positioning is used. In the other schemes, the author specifies the position: top, bottom, left, right.

**Layers** are specified by the z-index CSS property. It represents the third dimension of the box: its position along the "z axis".

The boxes are divided into stacks (called stacking contexts). In each stack the back elements will be painted first and the forward elements on top, closer to the user. In case of overlap the foremost element will hide the former element. The stacks are ordered according to the z-index property. Boxes with "z-index" property form a local stack.

## Trivia

### The birth of the web

Tim Berners-Lee, a British scientist at CERN, invented the World Wide Web (WWW) in 1989. The web was originally conceived and developed to meet the demand for automatic information-sharing between scientists in universities and institutes around the world.

The first website at CERN - and in the world - was dedicated to the World Wide Web project itself and was hosted on Berners-Lee's NeXT computer. The website described the basic features of the web; how to access other people's documents and how to set up your own server. The NeXT machine - the original web server - is still at CERN. As part of the project to restore [the first website](http://info.cern.ch/), in 2013 CERN reinstated the world's first website to its original address.

On 30 April 1993 CERN put the World Wide Web software in the public domain. CERN made the next release available with an open license, as a more sure way to maximize its dissemination. Through these actions, making the software required to run a web server freely available, along with a [basic browser](http://line-mode.cern.ch/) and a library of code, the web was allowed to flourish.

*More reading:*

[What really happens when you navigate to a URL](http://igoro.com/archive/what-really-happens-when-you-navigate-to-a-url/)

[How Browsers Work: Behind the scenes of modern web browsers](http://www.html5rocks.com/en/tutorials/internals/howbrowserswork/)

[What exactly happens when you browse a website in your browser?](http://superuser.com/questions/31468/what-exactly-happens-when-you-browse-a-website-in-your-browser)

[What happens when](https://github.com/alex/what-happens-when)

[So how does the browser actually render a website](https://www.youtube.com/watch?v=SmE4OwHztCc)

[Constructing the Object Model](https://developers.google.com/web/fundamentals/performance/critical-rendering-path/constructing-the-object-model)

[How the Web Works: A Primer for Newcomers to Web Development (or anyone, really)](https://medium.freecodecamp.com/how-the-web-works-a-primer-for-newcomers-to-web-development-or-anyone-really-b4584e63585c#.7l3tokoh1)
