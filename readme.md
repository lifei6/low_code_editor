# 前端低代码平台开发



# 技术栈

>vue3+jsx+scss+element-plus



# 功能

## 1.整体框架

分为四个区：

- 左侧物料区
- 顶部菜单栏
- 右侧属性控制栏目
- 中间的渲染区



## 2.实现注册组件和json代码块中key的映射关系

- 维护了一张映射表componentMap={key,component}

## 3.实现物料区组件的注册

- 采用闭包实现一个createEditorConfig函数返回内部的映射表和注册组件的方法register(component)
- component的数据结构为：{lable,preview,render,key} = {显示的标签，预览的组件渲染函数，需要渲染的组件渲染函数，类型}

## 4.实现初始页面状态的渲染

- 读取初始化的json数据，维护成一个响应式的转态state,根据state里面的blocks初始化渲染区
- 数据是双向绑定的修改state页面重新渲染，只要把我们物料区的东西拖拽过去，增加state中blocks的项目，会进行重新渲染更新页面，从而显示的是包含我们拖拽过去的组件的页面

## 5.实现物料区组件拖拽并在渲染区渲染

- 对物料区组件拖拽功能进行封装为一个useMenvDragger API：采用了h5的draggable

- 具体逻辑：

  1. 获取目标元素真实DOM

  2. 开始拖拽dragstart

  3. 拖拽期间效果(在dragstart事件里面设置)

     - dragenter进入元素触发,添加一个移动标识

       ```js
       e.dataTransfer.dropEffect = "move"
       ```

     - dragover在元素上移动时触发，必须阻止默认行为否则无法触发drop

       ```js
       e.preventDefault()
       ```

     - drapleave离开元素时触发，添加禁用标识

       ```js
       e.dataTransfer.dropEffect = "none"
       ```

     - drop执行放置，在放置目标上触发

       放置了元素生成拖拽元素真实DOM

  4. 拖拽结束dragend,清除拖拽效果的监听器

  5. 因为放置的位置是元素的左上角没有居中，拖拽完成后才生成了拖拽元素真实DOM，需要在挂载完成onMounted去设置放置居中效果

  > dragstart和dragend是在**拖拽元素**触发，dragenter/dragmove/dragleave/drop是在**目标元素**触发

## 6.实现渲染区组件的拖拽移动

- 先对渲染区组件的选中功能进行一个封装useFocus API 记录组件有哪些选中，哪些没有选中，记录选中的组件，可以单选也可以多选
- 对渲染区组件的拖拽进行一个封装useBlockDragger API
- 选中组件计算出当前的所有选中组件的位置{top,left}，根据这个去执行拖拽功能，以回调的形式执行

## 7.实现渲染区组件移动时的对齐辅助线

- 点击时先记录未选中的组件可能产生辅助线的所有情况
- 记录辅助线数据和这时的拖拽元素的位置{showTop,top}/{showLeft,left}
- 移动时根据距离辅助线是否小于5px进行快速贴近

## 8.实现菜单栏撤销与重做

> 主要思想采用消息订阅与发布，在初始化命令时进行发布start和end事件（发布）
>
> 在拖拽前触发start进行之前状态的记录，在拖拽后触发end事件（订阅）
>
> - 一方面记录拖拽后的状态
> - 一方面执行drag命令的执行函数，进行状态队列记录的增加
>
> 撤销与移动只需要在维护的状态队列中进行指针的移动，执行对应的记录状态的函数{mustdo:之前的状态，undo:之后的状态}就可以更新data,重新渲染页面，回到对应的之前的状态

- 实现物料区组件拖拽的撤销和重做
- 实现渲染区组件拖拽的撤销和重做

## 9.实现快捷键撤销与重做

- 在最开始就进行键盘事件的监听，事件触发时计算出当前按下的是字符串ctrl+z还是ctrl+y,接着遍历所有指令看是否有匹配的快捷键，有则执行对应的执行函数