## 前端测试指南

Marcus Efraimsson 于2018.8.14编辑本页面

### 通用
- *dir/a.ts* 组件的测试在 *dir/specs/a.test.ts*
- *React dir/a.tsx* 组件的测试在 *dir/a.test.tsx*
- 测试是单元测试而不应该是整个APP实例
- 从其他地方借鉴,最近写的测试用例
- 采用 *Jest* 编写
  
### 指南
* 纯函数比对象成员更容易测试.尝试编写相应地代码.
* 实例化类时,尝试在构造函数中提供最小化的
* 保持测试尽可能与框架无关,e.g.,:
  * 移除 *Angular* 专用的 *mocks*   
  * 用 *Promise.resolve()* 替换 *ctx.$q.when()*
  * 使用 *q* 包来处理 *.finally()* 是 *q.when()* 问题的情况

### 结构
* 执行函数将在测试块中测试,不是在 *setup* 或者 *beforeEach  *
* 弄清楚在测试什么:
```
import MyComponent, { processData } from '../MyComponent';

describe('MyComponent', () => {
  // Component-wide tests
  it('should do something', () => {
  });

  // Component-member tests
  describe('modifyComponent()', () => {
    it('should modify the component like this', () => {
      const component = new MyComponent();

      let options = {};
      component.modifyComponent(options);
      expect(component.x).toBe(..);

      options = { foo: 42 };
      component.modifyComponent(options);
      expect(component.x).toBe(..);
    });
  });
});

// Pure functions
describe('processData()', () => {
  it('should transform input to output', () => {
  });
});
```
### 例子
设置好 *stubs* 并将它们作为参数传递给构造函数:
* https://github.com/grafana/grafana/blob/master/public/app/features/dashboard/specs/change_tracker.jest.ts

这些测试使用简单的 *backend_srv stub* (https://github.com/grafana/grafana/blob/master/public/test/mocks/backend_srv.ts):
* https://github.com/grafana/grafana/blob/master/public/app/containers/ManageDashboards/FolderSettings.jest.tsx#L12
* https://github.com/grafana/grafana/blob/master/public/app/containers/AlertRuleList/AlertRuleList.jest.tsx#L13

测试使用 *Jest mock* 来测试是否发出了一个有时可能有用的 *event*.大多数情况下你应该使用 *stub* 而不是 *mock*,但也有必须使用 *mock* 的情况:
* https://github.com/grafana/grafana/blob/master/public/app/core/specs/search_results.jest.ts
