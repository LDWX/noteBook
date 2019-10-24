### 给饼状图每个区域添加指定颜色



```javascript
option = {
    title : {
        text: '全局指标状态分布图',
        x:'left',
        y:'top'
    },
    tooltip : {
        show: true,
        formatter: "{a} <br/>{b} : {c} ({d}%)"
    },
    color:['red', 'green','yellow','blueviolet']		//通过全局color属性配置每个区域的颜色
}

```

