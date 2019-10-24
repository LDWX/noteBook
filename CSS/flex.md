### 自适应宽度，并保持最后一行缩放比例

功能说明：所有方块随页面大小进行缩放，并且，最后一行与前面行保持相同缩放比例。

示意图：

![1566789905890](C:\Users\shenxin\AppData\Roaming\Typora\typora-user-images\1566789905890.png)

html:

```html
<div class="tab-content">
    <div
        v-for="(tree, index) in tmpTreeList" :key="index"
        class="content-item">{{tree.treeName}}</div>
    <div class="content-item not-show"></div>
    <div class="content-item not-show"></div>
    <div class="content-item not-show"></div>
    <div class="content-item not-show"></div>
</div>
```

想要实现最后一行的对齐，可以在数组后面**添加空元素**，flex在换行时，会自动忽略空元素，不会为空元素换行。

css:

```css
.tab-content {
    display: flex;
    flex-flow: row wrap;

    margin-top: 20px;
    text-align: left;
    white-space: nowrap;

    .content-item {
        flex: 1 1 200px;
        overflow: hidden;
        text-overflow: ellipsis;
        padding: 7px 15px;
        text-align: center;
        margin-right: 20px;
        margin-bottom: 20px;
        border: 1px solid rgba(0, 0, 0, .1);
        border-radius: 2px;

        +.content-item {
            &.active {
            border-color: #00bdd6;
            color: #00bdd6;
            }
        }
        &.not-show {
            width: 0;
            height: 0;
            padding: 0;
            border: 0;
        }
    }
}
```

