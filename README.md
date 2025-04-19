# 金币中心-获取记录

这是一个基于Figma设计稿实现的金币交易记录页面。

## 项目结构

```
/
├── index.html     // 主HTML文件
├── style.css      // 样式文件
├── images/        // 图片资源目录
│   ├── back_arrow.svg
│   ├── cellular_icon.svg
│   ├── coin_icon.png
│   └── wifi_icon.svg
└── README.md      // 说明文档
```

## 如何使用

1. 确保所有文件都位于同一目录中，保持上述的目录结构
2. 使用浏览器打开`index.html`文件，即可查看页面效果
3. 为了获得最佳的移动设备显示效果，请在浏览器的移动设备模拟模式下查看

## 页面特点

- 响应式设计，适配移动设备屏幕
- 模拟iOS状态栏和底部Home指示器
- 精确还原Figma设计稿中的金币交易记录列表
- 使用CSS实现模糊效果和精细的视觉细节

## 自定义

如需添加更多交易记录，只需在HTML文件的`transaction-list`区域内复制`transaction-item`结构并修改相应内容即可。 