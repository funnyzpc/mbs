
### 20200102_使用node+puppeteer+express搭建截图服务.md

#### 写在之前
  一开始我们的需求是打开报表的某个页面然后把图截出来，然后调用企业微信发送给业务群
  这中间我尝试了多种技术，比如`html2image`，`pdf2image`、`selenium`这些，这其中截图
  比体验较好的也就`selenium`了，不过我们有些页面加载的时间较长，selenium似乎对html互操作性
  也不是很完美(通过Thread.sleep并不能完美的兼容绝大多数报表)，另外还有一个比较要命的
  是Chromium渲染出来的页面似乎也有不同程度的问题(就是不好看),当然后面一个偶然的机会在
  某不知名网站看到有网友用`puppeteer`来实现截图，遂~，一通骚操作就搭了一套出来(虽然最终方案并不是这个
  ,当然这是后话哈～)，这里就拿出来说说哈～
  
#### 准备
由于整个系统是基于node+express的web服务，puppeteer只是node的一个plugin，所以需要做的准备大致有下
+ 一台linux服务器，这里实用centos
+ node安装包(用于搭建node环境)
+ 字体文件


#### 安装node环境
+ `wget https://nodejs.org/dist/v14.15.3/node-v14.15.3-linux-x64.tar.xz`
+ `tar --strip-components 1 -xvJf node-v* -C /usr/local`
+ `npm config set registry https://registry.npm.taobao.org`
  
#### 安装pm2(用于守护node服务)
__注意安装pm2前必须安装npm，如果只是非正式环境可以不用安装pm2__
+ `npm install pm2 -g`
+ 其它操作请见[https://pm2.keymetrics.io](https://pm2.keymetrics.io)

#### 安装字体
__这个其实很重要，我也绕了弯，原本以为改改字体编码就可以了，后来发现不是__
+ step1: 将window字体复制到linux下
  - windows: C:\Windows\Fonts
  - Linux: /usr/share/fonts/
+ step2: 建立字体索引信息并更新字体缓存
  - cd /usr/share/fonts/
  - mkfontscale
  - mkfontdir
  - fc-cache  

#### 准备代码
+ index.js
```
// 引入express module
// 引入puppeteer module
const express = require('express'),
    app = express(),
    puppeteer = require('puppeteer');

// 函数::页面加载监控
const waitTillHTMLRendered = async (page, timeout = 30000) => {
  const checkDurationMsecs = 1000;
  const maxChecks = timeout / checkDurationMsecs;
  let lastHTMLSize = 0;
  let checkCounts = 1;
  let countStableSizeIterations = 0;
  const minStableSizeIterations = 3;

  while(checkCounts++ <= maxChecks){
    let html = await page.content();
    let currentHTMLSize = html.length;
    let bodyHTMLSize = await page.evaluate(() => document.body.innerHTML.length);
    console.log('last: ', lastHTMLSize, ' <> curr: ', currentHTMLSize, " body html size: ", bodyHTMLSize);
    if(lastHTMLSize != 0 && currentHTMLSize == lastHTMLSize)
      countStableSizeIterations++;
    else
      countStableSizeIterations = 0; //reset the counter

    if(countStableSizeIterations >= minStableSizeIterations) {
      console.log("Page rendered fully..");
      break;
    }
    lastHTMLSize = currentHTMLSize;
    await page.waitFor(checkDurationMsecs);
  }
};

//创建一个 `/screenshot` 的route
app.get("/screenshot", async (request, response) => {
  try {
        const browser = await puppeteer.launch({ args: ['--no-sandbox'] });
        const page = await browser.newPage();
        await page.setViewport({
                            width:!request.query.width?1600:Number(request.query.width),
                            height:!request.query.height?900:Number(request.query.height)
                                                        });
        // 这里执行登录操作(非公共页面需要登录)
        if(request.query.login && request.query.login=="true"){
                // wait until page load
                await page.goto('认证(登录)地址', { waitUntil: 'networkidle0' });
                await page.type('#username', '登录用户名');
                await page.type('#password', '登录密码');
                // click and wait for navigation
                await Promise.all([
                        page.click('#loginBtn'),
                        page.waitForNavigation({ waitUntil: 'networkidle0' }),
                ]);
        }
        await page.goto(request.query.url,{'timeout': 12000, 'waitUntil':'load'});
        await waitTillHTMLRendered(page);
    const image = await page.screenshot({fullPage : true,margin: {top: '100px'}});
    await browser.close();
    response.set('Content-Type', 'image/png');
    response.send(image);
  } catch (error) {
    console.log(error);
  }
});
// listener 监听 3000端口
var listener = app.listen(3000, function () {
  console.log('Your appliction is listening on port ' + listener.address().port);
});
```
+ package.json
```
{
  "name": "funnyzpc",
  "version": "1.0.0",
  "description": "",
  "main": "index.js",
  "scripts": {
    "test": "echo \"Error: no test specified\" && exit 1"
  },
  "author": "",
  "license": "ISC"
}

```

#### 依赖安装
+ `npm i --save puppeteer express`

  [注意：如果安装失败 请检查是否更改为taobao源]

#### 启动及管理
+ 直接使用node启动服务
  - `node index.js`
+ 使用pm2启动(如果安装了pm2)
  - 启动：`pm2 start index.js`
  - 进程：`pm2 list`
  - 删除：`pm2 delete 应用ID`
  
#### 使用
由于以上代码已经对截图的加载做过处理的，所以无需在使用线程睡眠
同时代码也对宽度(width)和高度(height)做了处理，所以具体访问地址如下

`http://127.0.0.1:3000/screenshot/?width=[页面宽度]&height=[页面高度]&url=[截图地址]`

#### 最后
虽然我们我们使用`puppeteer`能应对绝大多数报表，后来发现`puppeteer`对多组件图表存在渲染问题，所以就要求
提供商提供导出图片功能(用户页面导出非api)，所以最终一套就是 http模拟登录+调用截图接口+图片生成监控+推送图片
好了，关于截图就分享到这里了，各位元旦节快乐哈～《^。^》
