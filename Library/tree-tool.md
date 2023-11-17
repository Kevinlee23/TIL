### 使用 tree-tool 工具库来处理树形结构

工具库资料: https://wintc.top/article/26
github: https://github.com/wintc23/js-tree-tool

#### 基本原理

##### 结构

```javascript
// 树结构
// 使用储存了树根节点的【列表】表示一个树形结构
let tree = [
  {
    id: "1",
    title: "节点1",
    children: [
      { id: "1-1", title: "节点1-1" },
      { id: "1-2", title: "节点1-2" },
    ],
  },
  {
    id: "2",
    title: "节点2",
    children: [{ id: "2-1", title: "节点2-1" }],
  },
];
```

##### 树的遍历

树结构遍历
---- 广度优先（循环实现）
---- 深度优先
----- 先序遍历（递归、循环实现）
----- 后序遍历（递归、循环实现）

```javascript
// 广度优先
function treeForeach(tree, func) {
  let node,
    list = [...tree];
  while ((node = list.shift())) {
    func(node); // 访问

    // 如果有子节点, 解构添加到列表最后
    node.children && list.push(...node.children);
  }
}

// 输出
// 节点1
// 节点2
// 节点1-1
// 节点1-2
// 节点2-1
```

```javascript
// 深度优先 - 先序遍历(递归实现)
function treeForeach2(tree, func) {
  tree.forEach((node) => {
    func(node);
    data.children && treeForeach2(node.children, func);
  });
}

// 输出
// 节点1
// 节点1-1
// 节点1-2
// 节点2
// 节点2-1

// 深度优先 - 后序遍历(递归实现)
function treeForeach3(tree, func) {
  tree.forEach((node) => {
    data.children && treeForeach3(node.children, func);
    func(node);
  });
}

// 输出
// 节点1-1
// 节点1-2
// 节点1
// 节点2-1
// 节点2
```

```javascript
// 深度优先 - 先序(循环实现)
function treeForeach4(tree, func) {
  let node,
    list = [...tree];
  while ((node = list.shift())) {
    func(node);
    // 将子节点加到列表最前面
    node.children && list.unshift(...node.chilren);
  }
}

// 深度优先 - 后序(循环实现)
function treeForeach(tree, func) {
  let node,
    list = [...tree],
    i = 0;
  while ((node = list[i])) {
    let childCount = node.children ? node.children.length : 0;
    // 没有子节点或子节点遍历完, 访问根节点, 并且往后遍历
    if (!childCount || node.children[childCount - 1] === list[i - 1]) {
      func(node);
      i++;
    } else {
      // 将子节点移到根节点的前面
      list.splice(i, 0, ...node.children);
    }
  }
}
```

#### 列表与树互转

##### 列表 -> 树

```javascript
// 结构
let list = [
  {
    id: "1",
    title: "节点1",
    parentId: "",
  },
  {
    id: "1-1",
    title: "节点1-1",
    parentId: "1",
  },
  {
    id: "1-2",
    title: "节点1-2",
    parentId: "1",
  },
  {
    id: "2",
    title: "节点2",
    parentId: "",
  },
  {
    id: "2-1",
    title: "节点2-1",
    parentId: "2",
  },
];

// 列表转树
function listToTree(list) {
  let info = list.reduce((map, node) => {
    map[node.id] = node;
    node.children = [];
    return map;
  }, {});

  return list.filter((node) => {
    info[node.parentId] && info[node.parentId].children.push(node);
    return !node.parentId;
  });
}
```

##### 树 -> 列表

```javascript
// 递归实现
function treeToList(tree, result = [], level = 0) {
  tree.forEach((node) => {
    result.push(node);
    node.level = level + 1;
    node.children && treeToList(node.children, result, level + 1);
  });

  return result;
}

// 循环实现
function treeToList2(tree) {
  let node,
    // 遍历根节点, level值为1
    result = tree.map((node) => ((node.level = 1), node));

  for (let i = 0; i < result.length; i++) {
    // 如果节点没有子节点, 跳过
    if (!result[i].children) continue;
    // 对所有子节点附加level值
    let list = result[i].children.map(
      (node) => ((node.level = result[i].level + 1), node)
    );

    // 将子节点插到它的父节点后面
    result.splice(i + 1, 0, ...list);
  }

  return result;
}
```

#### 其他功能性方法

##### 树结构筛选

```javascript
function treeFilter(tree, func) {
  // 使用map复制一下节点, 避免修改到原树
  return tree
    .map((node) => ({ ...node }))
    .filter((node) => {
      node.children = node.children && treeFilter(node.children, func);

      return func(node) || (node.children && node.children.length);
    });
}
```

##### 树结构查找

```javascript
// 查找节点
function treeFindNode(tree, func) {
  for (const node of tree) {
    if (func(node)) return node;
    if (node.children) {
      const res = treeFindNode(data.children, func);
      if (res) return res;
    }
  }

  return null;
}

// 查找节点路径
function treeFindPath(tree, func, path = []) {
  if (!tree) return [];
  for (const node of tree) {
    path.push(node.id);
    if (func(node)) return path;
    if (node.children) {
      const findChildren = treeFindPath(node.children, func, path);
      if (findChildren.length) return findChildren;
    }

    // 回溯
    path.pop();
  }

  return [];
}

// 查找多条路径
function treeFindMorePath(tree, func, path = [], result = []) {
  for (const node of tree) {
    path.push(node.id);
    func(data) && result.push([...path]);
    data.children && treeFindMorePath(node.children, func, path, result);

    // 回溯
    path.pop();
  }

  return result;
}
```
