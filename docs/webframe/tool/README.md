[[toc]]
## 常用软件下载
- teamviewer破解版 远程控制软件(附带破解视频教学) 提取码：7aud[下载](https://pan.baidu.com/s/1O_9hBfqq1vBLkx9E51RrWA) 
- centOS mac版本[下载](https://pan.baidu.com/s/1geK2kF5)
- postman破解版 接口调试工具 提取码：t5e9 [下载](https://pan.baidu.com/s/1FB82YFv6r2eSvj-5O3nczA)
- git win_x64 提取码：v3f1 [下载](https://pan.baidu.com/s/112SCA8KeS2Up6mekDl1uGw) 
- git win_32 提取码：01fk [下载](https://pan.baidu.com/s/1tMG-7agcfELfcbzBIsC2hQ) 
- navicat for mysql10.0.11简体中文破解版 提取码：z59z [下载](https://pan.baidu.com/s/1udENOBe6P_KQ7d8fyMBR6A 
- axureRP 9 破解版 提取码：t7jh [下载](https://pan.baidu.com/s/164DU5VoB8hYxqoT-QQd8Wg)


## 交流群讨论问题整理
- [Vuepress 如何引入百度统计和谷歌统计]()

方式1：在`config.js`中,配置head以及pligins：

```javascript
  head: [
        ['link', { rel: 'icon', href: '/image/favicon.ico' }],
        ['script', {}, `
            var _hmt = _hmt || [];
            (function() {
            var hm = document.createElement("script");
            hm.src = "https://hm.baidu.com/hm.js?**********************";
            var s = document.getElementsByTagName("script")[0]; 
            s.parentNode.insertBefore(hm, s);

            // 引入谷歌,不需要可删除这段
            var hm1 = document.createElement("script");
            hm1.src = "https://www.googletagmanager.com/gtag/js?id=UA-00000000-1";
            var s1 = document.getElementsByTagName("script")[0]; 
            s1.parentNode.insertBefore(hm1, s1);
            })();

            // 谷歌加载,不需要可删除
            window.dataLayer = window.dataLayer || [];
            function gtag(){dataLayer.push(arguments);}
            gtag('js', new Date());

            gtag('config', 'UA-00000000-1');
        `]
    ],
    pligins:[
        [
             '@vuepress/google-analytics',
          {
            'ga': 'UA-149666038-1' // UA-00000000-0
          }
        ]
    ]
```


方式2：在组件中引入， 目前只有百度有效：
首先安装`vue-ba`,

```javascript
 import("vue-ba").then(module => {
      let ba = module.default;
      Vue.use(ba, "");
      Vue.use(ba, { siteId: "" });
    });
```