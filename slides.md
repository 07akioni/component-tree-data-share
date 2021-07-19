---
# try also 'default' to start simple
theme: default
class: 'text-center'
# https://sli.dev/custom/highlighters.html
highlighter: shiki
# some information about the slides, markdown enabled
info: 关于组件库树形数据的分享
---

# 组件库树形数据结构的秘密

07akioni

---

# 关于我

<div style="display: flex; padding-top: 24px; align-items: flex-start;">

<img src="https://07akioni.oss-cn-beijing.aliyuncs.com/07akioni.jpeg" style="width: 20%; border: 2px solid #aaa; border-radius: 4px; margin-right: 24px;" >

<div>

- 07akioni
  - 写网页的
  - 写过一个组件库
  - 写过一些乱七八糟的库
    - 总的来说还是写网页的
  - 北京

</div>

</div>

---

# 目录

&nbsp;

- 树形数据使用场景
- 如何获取树形数据
- 涉及的特性
- 特性的常见实现
- 特性的统一实现
-  一点性能优化

---

# 目录

&nbsp;

- **树形数据使用场景**
- 如何获取树形数据
- 涉及的特性
- 特性的常见实现
- 特性的统一实现
-  一点性能优化

---

# 组件库中的树形结构使用场景

&nbsp;

**哪里会用到?**

<v-click>

- Tree 组件
- Menu 组件

</v-click>

<v-click>

**有没有其他的？**

</v-click>

<v-click>

- Table 组件
- Dropdown 组件
- Cascader 组件
- TreeSelect 组件

</v-click>

<v-click>

**还有么？**

</v-click>

<v-click>

- Select - 数组也是树

</v-click>

---

# 目录

&nbsp;

- 树形数据使用场景
- **如何获取树形数据**
- 涉及的特性
- 特性的常见实现
- 特性的统一实现
-  一点性能优化

---

# 如何获取树形数据

### Props API vs Children API

<div class="grid grid-cols-2 gap-x-4 mb-4">

<div>

Props API

```jsx
<Tree
  data={[
    { label: 'A', key: 'A' },
    {
      label: 'B',
      key: 'B',
      children: [
        {
          label: 'C',
          key: 'C'
        }
      ]
    }
  ]}
/>
```

</div>

<div>

Children API

```jsx
<Tree>
  <TreeNode key="A">A</TreeNode>
  <TreeNode key="B">
    B<TreeNode key="C">C</TreeNode>
  </TreeNode>
</Tree>
```

<v-click>

😊 看起来更简单，数据和 JSX 结构一致，label 好写

</v-click>

<v-click>

😕 但是出来混总是要还的

- 最终 `JSX.Element` 还是要被转化成 data
- 不然怎么知道谁先谁后
- `children` => `data`，难写，性能差

</v-click>

</div>

</div>

---

# 如何获取树形数据

&nbsp;

😊 看起来更简单，数据和 JSX 结构一致，label 好写

- 支持这两个会给你带来很多开发上的麻烦
- 如果你在用户、老板的胁迫下，或者觉得好处更多增加了这类 API，做好花更多时间维护的准备

😕 但是出来混总是要还的

- 最终 `JSX.Element` 还是要被转化成 data（不然怎么知道谁先谁后，谁是父级谁是子级）
- `children` => `data`，难写，性能差
- 你的用户会来问你为啥 `TreeNode` 不能继续封装
- 性能不好优化
  - React：每次创建很多 ReactElement
  - Vue：收集过程避免触发依赖

---

# 目录

&nbsp;

- 树形数据使用场景
- 如何获取树形数据
- **涉及的特性**
- 特性的常见实现
- 特性的统一实现
-  一点性能优化

---

# 树形数据涉及的特性

| **名称**   | **场景**                                             |
| ---------- | ---------------------------------------------------- |
| 查询节点   | 通过 key 查询                                        |
| 勾选       | 非级联选择；级联选择                                 |
| Group 节点 | 把一些选项合为一组                                   |
| 移动       | 父、子、兄弟；在成组节点中；循环；跳过 Disabled 节点 |
| 折叠       | 折叠展开                                             |
| 打平节点   | 虚拟列表                                             |
| 非数据节点 | divider                                              |
| 获取路径   | 高亮选中的 item                                      |

---

# 树形数据涉及的特性

| **组件**   | **勾选** | **移动** | **成组** | **折叠** | **非数据节点** | **打平节点** | **查询节点** | **获取路径** |
| ---------- | -------- | -------- | -------- | -------- | -------------- | ------------ | ------------ | ------------ |
| Tree       | ✅       | ✅       | ✅       | ✅       | ❌             | ✅           | ✅           | ❌           |
| Menu       | ❌       | ❌       | ✅       | ✅       | ❌             | ❌           | ✅           | ✅           |
| Table      | ✅       | ❌       | ❌       | ✅       | ❌             | ✅           | ✅           | ❌           |
| Dropdown   | ❌       | ✅       | ✅       | ❌       | ✅             | ❌           | ✅           | ✅           |
| Cascader   | ✅       | ✅       | ❌       | ❌       | ❌             | ❌           | ✅           | ✅           |
| TreeSelect | ✅       | ✅       | ✅       | ✅       | ❌             | ✅           | ✅           | ❌           |

---

# 目录

&nbsp;

- 树形数据使用场景
- 如何获取树形数据
- 涉及的特性
- **特性的常见实现**
- 特性的统一实现
-  一点性能优化

---

# 实现特性的常见方法

针对于每种组件定制特定的数据结构

<div class="grid grid-cols-2 gap-x-4 mb-4">

<div>

### Tree

```ts
export interface DataNode {
  checkable?: boolean
  children?: DataNode[]
  // ...
}
```

### Cascader

```ts
export interface CascaderOption {
  value?: string | number
  label?: React.ReactNode
  // ...
}
```

各自为政，维护困难

</div>

<div>

### Select

```ts
export interface OptionCoreData {
  key?: Key
  disabled?: boolean
  // ...
}
```

### Menu

```ts
export interface MenuInfo {
  key: string
  keyPath: string[]
  // ...
}
```

</div>

</div>

---

# 目录

&nbsp;

- 树形数据使用场景
- 如何获取树形数据
- 涉及的特性
- 特性的常见实现
- **特性的统一实现**
-  一点性能优化

---

# 使用同一种数据结构处理

&nbsp;

**这种数据结构需要满足：**

- 不修改原有数据
- 每个节点和原有数据一一映射
- 每个节点可以找回原有数据节点

**假设我们有一个很强大的方法 `createXTree`**

- 可以创造出一个 `xTree` 实例
- `xTree` 中的节点叫 `xNode`，和原有数据节点一一对应
- `xTree` 实现了所有之前提到的特性

---

# `xTree` 实现的特性

| **名称**       | **方法**                                              |
| -------------- | ----------------------------------------------------- |
| _可追溯原数据_ | `xNode.rawNode` 即对应原始节点                        |
| 查询节点       | `xTree.getNode(key)`                                  |
| Group 节点     | `{ type: 'group', label: '', children: [] }`          |
| 非数据节点     | `{ type: 'ignored' }`                                 |
| 获取路径       | `xNode.getPath()`                                     |
| 移动           | `xNode.getPrev()`, `xTree.getPrev(key)`, ...          |
| 折叠           | `xTree.getFlattenedNodes(collapsedKeys)`              |
| 打平节点       | `xTree.getFlattenedNodes(collapsedKeys)`              |
| 勾选           | `xTree.getCheckedKeys(checkedKeys, cascade: boolean)` |

---

# 特性实现：可追溯原数据

树的遍历

```js
function createXTree(data) {
  const xTree = {
    nodes: []
    // ...
  }
  function traverse(data) {
    // 向 xTree.nodes 添加 xNode
    // xNode.rawNode 设为原始 node
    xNode.rawNode = originalNode
  }
  traverse(data)
  return xTree
}
```

---

# 特性实现：查询节点

查询，建立使用 Map

```js
function createXTree(data) {
  const xTree = {
    nodes: [],
    map: new Map()
  }
  function traverse(data) {
    // 向 xTree.nodes 添加 xNode
    // xNode.rawNode 设为原始 node
    xNode.rawNode = originalNode
    // 遍历的过程将 xNode 添加到 xTree.map
    xTree.map.set(rawNode.key, xNode)
  }
  traverse(data)
  return xTree
}
```

---

# 特性实现：Group 节点和非数据节点

为 `xNode` 建立相关属性

```js
const xNode = {
  rawNode,
  isIgnored() {
    return this.rawNode.type === 'ingored'
  },
  isGroup() {
    return this.rawNode.type === 'group'
  }
}
```

之后会介绍为什么用 `getter`，无伤大雅

---

# 特性实现：移动

关联 `xNode`，在建立 `xTree` 的过程中为 `xNode` 添加属性

```js
function createXTree(data) {
  // ...
  function traverse(data) {
    // 向 xTree.nodes 添加 parent, children, siblings
    xNode.parent = traverseParent
    xNode.children = traverseChildren
    xNode.siblings = traverseSiblings
  }
  traverse(data)
  return xTree
}
```

```js
const xNode = {
  // return parent
  getParent() {},
  // loop xNode.children
  getFirstAvailableChild() {},
  // loop xNode.siblings
  getNext() {}
}
```

---

# 特性实现：获取路径

一个低级的移动

```js
const xNode = {
  getPath() {
    const path = [this]
    while (this.parent) {
      path.push(this.parent)
    }
    return path.reverse()
  }
}
```

---

# 特性实现：折叠 + 打平节点

用于虚拟列表的实现

依然使用树的遍历

```js
function flatten(nodes, collapsedKeys) {
  const flattenedNodes = []
  function traverse(nodes) {
    nodes.forEach((node) => {
      flattenedNodes.push(node)
      if ('children' in node && collapsedKeys.includes(node.key)) {
        traverse(node.children)
      }
    })
  }
  traverse(nodes)
  return flattenedNodes
}
```

---

# 特性实现：勾选

<div style="display: flex;">

<div>

**关于勾选**

- 重点在于 Cascade Check
- 如果你用过 antd，使用 `checkStrictly=false` 来开启这个 feature

**期望**

- 性能好，时间复杂度 `o(n)`
- 不影响原有数据结构（不直接在输入数据上修改）
- 支持不完整的异步数据（即使只有部分数据也可以正确的勾选）
- 处理 Disabled 节点
- 不重新生成整颗树
- 对于任意的输入，都能计算出正确的勾选节点

</div>

<div style="position: relative; display: flex; margin-right: 16px; width: 20%;">
  <img src="/cascade1.png" style="display: block; position: absolute; left: 0; right: 0; top: 0; bottom: 0;" />
  <img v-click src="/cascade2.png" style="display: block; position: absolute; left: 0; right: 0; top: 0; bottom: 0;" />
</div>

</div>

---

# 特性实现：勾选

&nbsp;

**直接计算一颗新的树？性能开销有点大**

<div class="grid grid-cols-2 gap-x-4 mb-4">

<div>

```js
const checkedKeys = [2]
const tree = [
  {
    key: 1,
    children: [
      {
        key: 2
      },
      {
        key: 3
      }
    ]
  },
  {
    key: 4
  }
]
```

</div>

<div>

```js
const newTree = [
  {
    key: 1,
    checked: 'half',
    children: [
      {
        checked: true,
        key: 2
      },
      {
        checked: false,
        key: 3
      }
    ]
  },
  {
    key: 4
  }
]
```

</div>

</div>

---

# 特性实现：勾选

&nbsp;

**希望直接拿到相关的 key**

<div class="grid grid-cols-2 gap-x-4 mb-4">

<div>

```js
const checkedKeys = [2]
const tree = [
  {
    key: 1,
    children: [
      {
        key: 2
      },
      {
        key: 3
      }
    ]
  },
  {
    key: 4
  }
]
```

</div>

<div>

```js
const newKeys = {
  checked: [2],
  halfChecked: [1]
}
```

<v-click>

🤔 能不能用常见的前序遍历计算？

</v-click>

<v-click>

- ❌ 自顶向下？不可，父级的状态依赖子级
- ❌ 子级还没计算，父级无法知道
- ❌ 父级依赖子级，递归套递归？性能差

</v-click>

<v-click>

🤔 自底向上呢？

</v-click>

<v-click>

- ❌ 也不够，子级的状态依赖祖先

</v-click>

</div>

</div>

---

# 特性实现：勾选

&nbsp;

<div style="display: flex; align-items: stretch; height: 54vh;">

<div>

<p style="margin-top: 0;">😎 真相只有一个，先自顶向下，再自底向上。</p>

<div v-click="2">

- 先将选中的 key 向下传播选中状态
  - 选中的 key 下面的节点全部选中
  - Disabled 节点不传播

</div>

<div v-click="4">

- 再将选中的 key 向上传播选中 + 半选中状态
  - 从最底层向上计算
  - 当前遍历的节点是否勾选（半勾选）通过下一层的勾选状态确定 - 因为自底向上传播，计算上层时下层已经计算完毕
  - Disabled 节点不传播

</div>

<div v-click="6">

- 🤔 那么每一层的节点哪里来的？
  - 在创建 `xTree` 的时候遍历储存的

</div>

</div>

<div style="position: relative; width: 35%; margin-left: 24px;">

<img style="position: absolute; left: 0; top: 0; height: 40vh;" src="/check1.png" />
<img v-click="1" style="position: absolute; left: 0; top: 0; height: 40vh;" src="/check2.png" />
<img v-click="3" style="position: absolute; left: 0; top: 0; height: 40vh;" src="/check3.png" />
<img v-click="5" style="position: absolute; left: 0; top: 0; height: 40vh;" src="/check4.png" />

</div>

</div>

---

# 目录

&nbsp;

- 树形数据使用场景
- 如何获取树形数据
- 涉及的特性
- 特性的常见实现
- 特性的统一实现
- ** 一点性能优化**

---

# 性能优化 1：利用 Getter 减少 Vue 中不必要的渲染

```js
export default {
  name: 'Tree',
  setup(props) {
    return {
      xTree: computed(() => createXTree(props.treeData))
    }
  },
  render() {
    // 使用 xTree 渲染树
  }
}
```

假设有某一个节点的 `disabled` 更改，不想触发树的渲染。

`createXTree` 中不能访问原有数据节点的 `disabled`。

---

# 性能优化 1：利用 Getter 减少 Vue 中不必要的渲染

```js
function createXTree() {
  const xNode = {
    disabled: node.disabled // 不可
  }
}
```

**使用 getter**

哪里渲染哪里访问

```js
function createXTree() {
  const xNode = {
    get disabled() {
      return node.disabled
    }
  }
}
```

---

# 性能优化 2：利用 Object.create 优化节点创建性能

&nbsp;

**一个节点**

```js
const xNode = {
  key: '',
  getPrev() {},
  getNext() {}
}
```

在节点挂方法，每个节点实例都需要属性吗？答案是否定的。

---

# 性能优化 2：利用 Object.create 优化节点创建性能

&nbsp;

**降低内存占用**

```js
const node = {
  key: ''
}

Object.setPrototypeOf(node, {
  getPrev() {},
  getNext() {}
})

// 或

node.__proto__ = {
  getPrev() {},
  getNext() {}
}
```

---

# 性能优化 2：利用 Object.create 优化节点创建性能

&nbsp;

**提升性能**

```js
const node = Object.create({
  getPrev() {},
  getNext() {}
})

node.key = ''
```

<div class="grid grid-cols-2 gap-x-4 mb-4">

<div>

### Object.setPrototypeOf

```js
const b = { b: 'b' }
for (let i = 0; i < 1000; ++i) {
  const a = { a: 'a' }
  Object.setPrototypeOf(a, b)
}
```

`1710.3 ops/s 慢 95.77%`

</div>

<div>

### Object.create

```js
const b = { b: 'b' }
for (let i = 0; i < 1000; ++i) {
  const a = Object.create(b)
  a.a = 'a'
}
```

`40397.54 ops/s`

</div>

</div>

---

# TypeScript

方法这么多，怎么区分节点

```ts
const tree = [] // 树数据
const xTree = createXTree(tree) // 处理过后的树

const xNode = xTree.get(key) // 不想要 Group Node 和 Ignored Node
```

利用泛型。

```ts
type CreateXTree<
  NormalNode = any,
  GroupNode = NormalNode,
  IgnoredNode = NormalNode
> = {
  get(key: string | number): NormalNode | null
}
const xTree = createXTree<NormalNode, GroupNode, IgnoredNode>(tree)

xTree.get() // NormalNode
```

---

# 实现（TreeMate）

TreeMate 实现了所有特性

一个组件库只需要一种树形数据结构。

https://github.com/07akioni/treemate

```js
import { createTreeMate } from 'treemate'

const treeMate = createTreeMate(treeData)
```

<v-click>

毕竟我讲了这么多也不是指望大家去实现。

</v-click>