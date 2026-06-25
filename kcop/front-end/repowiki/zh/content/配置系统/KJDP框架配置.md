# KJDP 框架配置

`loader-frame-config.js` 声明 KUI 框架配置：

```javascript
module.exports = {
    name: "kui",
    version: "4.0.3",
    frame: ["frame/lib", "frame/core", "frame/kui"],
    themeConf: {
        default: "blue",
        "smart-blue": { url: "frame/themes/smart-blue.css#" },
        "deep-blue": { url: "frame/themes/deep-blue.css#" },
        "blue": { url: "frame/themes/blue.css#" }
    },
    adaptKdop: true,
    isKdop: true
};
```

维护时不要轻易改变 `frame` 加载顺序。
