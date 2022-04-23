+++
author = "zs"
title = "优化长列表渲染及滚动"
date = "2022-04-05"
lastmod = "2022-04-05"
description = "Optimised long list rendering and scrolling"
tags = [
    "optimise",
]
+++

>需求：

长列表/大数据量表格渲染慢，滚动卡顿；1000条30个数据列的表格会有明显卡顿；

>解决方法：

**[antd-table-virtual-list](https://ant.design/components/table-cn/#components-table-demo-virtual-list)** ：antd-table提供的虚拟列表方案无法同时实现「可展开/选择/拖拽」等功能；

**[virtuallist-antd](https://github.com/crawler-django/virtuallist-antd)**：可以快速地在原代码的基础上实现优化效果，改动较小，表格的扩展功能可以保留；（好像只支持函数组件的写法）

>Complex example：

需要优化的代码：（通过代码可以看到包含了较多业务逻辑）

```js
import 'Table' from 'antd';

<Table
  columns={columns} // 经过处理的
  dataSource={dataSource} // 已处理过的数据
  onExpand={this.onExpand} // 可展开的回调函数
  expandedRowKeys={expandedRowKeys}
  rowKey={(record: any) => record.id}
  scroll={{y: '55vh'}}
  pagination={false}
  loading={loading}
  onChange={this.handleTableChange} // 用于监听数据列排序的函数
/>
```

>安装 **[virtuallist-antd](https://github.com/crawler-django/virtuallist-antd)**

`npm install --save virtuallist-antd`

>优化后：

```ts
// VisualTable.tsx
import React, { useMemo } from "react";
import { Table } from "antd";
import { VList } from "virtuallist-antd";

interface Props {
  columns: any;
  dataSource: any;
  expandedRowKeys: any;
  rowKey: any;
  scroll: any;
  pagination: any;
  loading: any;
  onExpand():any;
  onChange():any;
}
function VisualTable(props: Props) {

  const vc1 = useMemo(() => {
    return VList({
      height: '55vh',
      vid: 'first',
      resetTopWhenDataChange: false // 当数据改变时是否回滚顶部
    })
  }, []);

  return (
    <>
      {/* 所有之前业务逻辑通过props传入，基本不用做修改，特殊情况特殊处理 */}
      <Table
        columns={props.columns}
        dataSource={props.dataSource}
        pagination={props.pagination}
        onExpand={props.onExpand}
        expandedRowKeys={props.expandedRowKeys}
        /** 不建议使用x: max-content. 如果columns有fixed, x为max-content的话. ellipsis会失效 */
        scroll={{ y: '55vh', x: "100%" }}
        loading={props.loading}
        rowKey={props.rowKey}
        components={vc1}
        onChange={props.onChange}
      />
    </>
  )
}

export default VisualTable;
```

```
- import 'Table' from 'antd';
+ import 'VisualTable' from 'VisualTable.tsx';

- <Table
+ <VisualTable
    columns={columns} // 经过处理的
    dataSource={dataSource} // 已处理过的数据
    onExpand={this.onExpand} // 可展开的回调函数
    expandedRowKeys={expandedRowKeys}
    rowKey={(record: any) => record.id}
    scroll={{y: '55vh'}}
    pagination={false}
    loading={loading}
    onChange={this.handleTableChange} // 用于监听数据列排序的函数
  />
```

>问题：
因为点击展开后会修改dataSource，为了使点击展开后不回滚顶部，设置了resetTopWhenDataChange: false，但是切换帐期类型也会修改dataSource，导致不回滚顶部；

```javascript
import {scrollTo} from "virtuallist-antd";

// 需要的业务逻辑下使用，可回滚顶部；
scrollTo({row: 1, vid: '相应的vid'});
```



