# 前端：代码结构，架构和未来计划 [原文](https://github.com/grafana/grafana/wiki/Frontend:-Code-structure,-architecture-and-plans-for-the-future)
Marcus Efraimsson 于2018.8.14编辑本页面

## 前端
Grafana 的前端是单页应用, 采用 Angular, React 和 Sass 编写. 使用 webpack 构建. Angular 是处理URL->组件路由的主要框架.

## 结构
前端代码放在 public 目录下

| 位置 | 描述 |
| :------ | :------ |
| app/app.ts | 这个是 Angular 应用的自举代码 |
| app/sass | 前端样式 |
| app/core | 公共组件和服务 |
| app/features | 按功能来划分的组件和服务 |
| app/routes | URL -> Angular 或 React 的组件映射 |
| app/stores | 移动端存储 |
| app/plugins | 所有的内置插件(面板,数据源) |
| app/containers | React 容器和容器专用的组件 |

## 核心组件和服务
- backendSrv: 用于所有后端API请求和datasource请求
- templateSrv: 用于仪表盘组件来插入一个字符串包含模板变量
- timeSrv: 用于仪表盘组件调整时间区间
- dashboardSrv: 用于仪表盘的存取和加载
- bridgeSrv: 桥接 React 和 Angular (特别是URL状态的转换)
  
## 单元测试
我们重视清晰可读的代码,松耦合并且覆盖单元测试.我们使用Jest进行所有JavaScript测试.

详见[测试指南](https://github.com/grafana/grafana/wiki/Frontend-Test-Guidelines)

## React + Angular
我们正在迁移完整的基础代码到 React.这需要多年的努力.我们之所以做这项工作是x想确保 Grafana 保持现代化的前端应用,而且具有清晰可维护的代码.AngularJS 有很多问题让复杂 HTML 的渲染变得昂贵而且难以维护.同时,我们发现 React 是一个更简单的,易学易用的框架.它很大程度上会比 AngularJS 存活更久,而且最重要的是,如果我们使用这个框架而不是坚持永远使用 AngularJS 的话,它更容易吸引有才能的前端开发者.

目前,有一些整个页面都是用 React 编写的，部分组件也是用 React 编写的.在 Angular 中我们可以很容易的使用 React,但是反过来却有点棘手但也是可行的. 

## 进行中的重构/重写
- Angular -> React 重写

## 功能重构/重写
- 简化代码结构(将组件从 app/core 移到 app/components)
- 修复文件命名不一致的问题,Angular 和其他js组件使用蛇形命名法而 React 使用驼峰命名法
- 可以使用 React 编写 panel 和 datasource 插件 
- backendSrv 以及使用 Angular $http 的需要迁移到 fetch

## 未来的挑战
- 进行浏览器集成测试
- 插件回归测试(能够捕捉不同状态的面板的图像?)
- 通过 viewStateSrv 处理的仪表板 URL 状态会变的非常混乱

