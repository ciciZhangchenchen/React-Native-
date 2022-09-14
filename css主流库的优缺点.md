# css主流库的优缺点

## 官方用法：StyleSheet

StyleSheet 参考了CSS StyleSheets的类似的抽象写法
```jsx
import React from "react";
import { StyleSheet, Text, View } from "react-native";

const App = () => (
  <View style={styles.container}>
    <Text style={styles.title}>React Native</Text>
  </View>
);

const styles = StyleSheet.create({
  container: {
    flex: 1,
    padding: 24,
    backgroundColor: "#eaeaea"
  },
  title: {
    marginTop: 16,
    paddingVertical: 8,
    borderWidth: 4,
    borderColor: "#20232a",
    borderRadius: 6,
    backgroundColor: "#61dafb",
    color: "#20232a",
    textAlign: "center",
    fontSize: 30,
    fontWeight: "bold"
  }
});
export default App;
```

## 使用StyleSheet可以提高代码质量：

   通过将样式从render()函数中分离，可以让代码更加容易理解
   
   通过命名样式对于语义化低级组件（RN自提供的组件）是一个很好的方案
   
## 使用StyleSheet可以提高性能：

  使用StyleSheet.create()创建的样式可以通过ID重复引用，而内联样式则每次都需要创建一个新的对象

  它还允许仅通过桥(bridge)发送一次样式。 所有后续使用都将引用一个ID（尚未实现👈贡献RN的机会来啦）
## StyleSheet提供的一些API

### hairlineWidth
该常数始终是像素的整数倍（因此，由其定义的线看起来会很清晰(look crisp?)）并将尝试匹配基础平台上细线的标准宽度。但是您不应该依赖它作为一个恒定值使用，因为在不同平台和屏幕像素下，其值可能会以不同的方式计算。

### absoluteFill
一个非常常见的创建具有绝对位置和零位置的叠加层样式，因此可以使用 absoluteFill来方便并减少那些不必要的重复样式
对象已经被冻结，所以无法修改absoluteFill。

### absoluteFillObject
全屏弹层对象
```ts
interface AbsoluteFillStyle {
        position: 'absolute';
        left: 0;
        right: 0;
        top: 0;
        bottom: 0;
    }
```
### compose
组合两种样式，以便style2会覆盖style1中的同类型样式属性。

源码：
```js
function compose(style1, style2){
  if(style1 != null && style2 != null){
    return ([style1, style2])
  } else {
    return style1 != null ? style1 : style2;
  }
}
```
在DOM diff对比时样式属性则无法保持引用相等，所以需要使用StyleSheet.compose辅助函数方便实现

### flatten

扁平化一个样式对象数组，返回一个聚合的样式对象。
或者，可以使用此方法查找由StyleSheet.register返回的IDs。
该方法在内部使用StyleSheetRegistry.getStyleByID(style)来解析由ID表示的样式对象。 因此，将样式对象的数组（StyleSheet.create的实例）分别解析为它们各自的对象，合并为一个对象然后返回。 这也解释了替代用法。

### create
创建一个StyleSheet对象并引用给定的对象。

By the way,返回的对象也会被冻结不可被修改，所以建议在组件外部并使用const声明：

## RN的Color

在React-Native内支持以下5种颜色写法：

- rgb/rgba，如rgb(0, 0, 255)、rgba(0,0,255,1)
- 十六进制颜色(hex color)
  如"#F00"(#rgb)、#FF0000"(#rrggbb)、#f0ff"(#rgba)、#ff00ff00"(#rrggbbaa)
- 色调-饱和度-亮度(Hue-saturation-lightness)，
  如"hsl(360, 100%, 100%)"、"hsla(360, 100%, 100%, 1.0)"
- 透明度,rgab(0,0,0,0)的快捷写法："transparent"
- 称谓颜色(Named colors)，如"red"、"blue"，遵循CSS3 规范

但是做原生拓展时，需要传入的颜色属性往往是 int 单一类型，对于多种可能的颜色格式再在原生代码中格式化肯定是不现实的，
fb团队也考虑到了这一点，通过开放normalizeColor.js和processColor.js2个文件进行处理。
```js
// return 0xFF0000FF
normalizeColor("red");    
normalizeColor("#F00");
```

## styled-component

### 优点
- Automatic critical CSS:

   styled-components 持续跟踪页面上渲染的组件,并向自动其注入且仅注入样式. 结合使用代码拆分, 可以实现仅加载所需的最少代码.
   
- 解决了 class name 冲突:

   styled-components 为样式生成唯一的 class name. 开发者不必再担心 class name 重复,覆盖和拼写错误的问题.
   
- CSS 更容易移除

    想要确切的知道代码中某个 class 在哪儿用到是很困难的. 使用 styled-components 则很轻松, 因为每个样式都有其关联的组件. 如果检测到某个组件未使用并且被删除,则其所有的样式也都被删除.
    
- 简单的动态样式

    可以很简单直观的实现根据组件的 props 或者全局主题适配样式,无需手动管理数十个 classes.
    
- 无痛维护: 
 
  无需搜索不同的文件来查找影响组件的样式.无论代码多庞大，维护起来都是小菜一碟。
  
- 自动提供前缀:
 
 按照当前标准写 CSS,其余的交给 styled-components handle 处理.

- 支持的其他功能：
  1. 基于属性的适配，（某些条件下增加某个样式可以直接写在styled里面）
  2. 样式继承
  3. sass写法
  4. 主题切换

### 参考文档

[中文](https://github.com/hengg/styled-components-docs-zh)

[源码](https://github.com/styled-components/styled-components/blob/main/packages/styled-components/src/models/StyledNativeComponent.ts)




