---
title: hexo+github(二)
date: 2022-04-09 21:50:59
tags: hexo
---

 一 选择hexo自带的的图片插件

a:下载插件

> **hexo-asset-image**
>
> npm install hexo-asset-image --save

b：如果想添加图片到已经存在的md文件，那么这个时候直接自己创建一个与md文件同名的文件夹存放图片；

​      如果这个时候还没有创建md文件，可以选择hexo n 新建一个md文件，这个时候可以看到自动创建了一个与md同名的文件夹。

c：更改congif.yml文件里面的post_asset_folder: false更改为post_asset_folder: true。

d：更改node.module/hexo-asset-image下面的配置文件index.js为：

```
'use strict';
var cheerio = require('cheerio');

// http://stackoverflow.com/questions/14480345/how-to-get-the-nth-occurrence-in-a-string
function getPosition(str, m, i) {
  return str.split(m, i).join(m).length;
}

hexo.extend.filter.register('after_post_render', function(data){
  var config = hexo.config;
  if(config.post_asset_folder){
    var link = data.permalink;
    var beginPos = getPosition(link, '/', 3) + 1;
    var appendLink = '';
    // In hexo 3.1.1, the permalink of "about" page is like ".../about/index.html".
    // if not with index.html endpos = link.lastIndexOf('.') + 1 support hexo-abbrlink
    if(/.*\/index\.html$/.test(link)) {
      // when permalink is end with index.html, for example 2019/02/20/xxtitle/index.html
      // image in xxtitle/ will go to xxtitle/index/
      appendLink = 'index/';
      var endPos = link.lastIndexOf('.');
    }
    else {
      var endPos = link.lastIndexOf('/');
    }
    link = link.substring(beginPos, endPos) + '/' + appendLink;

    var toprocess = ['excerpt', 'more', 'content'];
    for(var i = 0; i < toprocess.length; i++){
      var key = toprocess[i];

      var $ = cheerio.load(data[key], {
        ignoreWhitespace: false,
        xmlMode: false,
        lowerCaseTags: false,
        decodeEntities: false
      });

      $('img').each(function(){
        if ($(this).attr('src')){
          // For windows style path, we replace '\' to '/'.
          var src = $(this).attr('src').replace('\\', '/');
          if(!(/http[s]*.*|\/\/.*/.test(src)
            || /^\s+\//.test(src)
            || /^\s*\/uploads|images\//.test(src))) {
            // For "about" page, the first part of "src" can't be removed.
            // In addition, to support multi-level local directory.
            var linkArray = link.split('/').filter(function(elem){
              return elem != '';
            });
            var srcArray = src.split('/').filter(function(elem){
              return elem != '' && elem != '.';
            });
            if(srcArray.length > 1)
            srcArray.shift();
            src = srcArray.join('/');

            $(this).attr('src', config.root + link + src);
            console.info&&console.info("update link as:-->"+config.root + link + src);
          }
        }else{
          console.info&&console.info("no src attr, skipped...");
          console.info&&console.info($(this));
        }
      });
      data[key] = $.html();
    }
  }
});

```

tips：在本人尝试时为成功，老师指导后知道，是因为源程序正则表达式有问题，在博客对图片进行检查时发现映射的地址为./com，这个地址并不是本地的地址，更改正则表达式后发现可以找到本地图片的地址。

可以在index.js中加入console.info语句打印变量，查看问题！！！！

f：重启hexo服务器

hexo s

如果重启后仍然看不到图片，可以按顺序运行以下代码：

> hexo clean
>
> hexo g
>
> hexo d
>
> hexo s



修改样式：

到github上下载主题

git clone 。。。。

然后将hexo/blog下的config文件中的theme的名称改为想要更改的主题的名称





hexo clean 清理缓存

hexo g 生成文件

hexo d 部署
