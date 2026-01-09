# antd 使用场景

## Form
> 其实最重要的是，要先确定是用 react hook + change 事件这样的方式去获取数据，还是直接用 antd 中提供的 form 来获取数据
> 切记，不能混用！！！
> 这两个最大的区别就是，原生的方式需要手动处理 change 事件，数据的更新; 采用 antd 中的方式不用关心数据的绑定、更新，直接获取即可

**获取表单数据**
+ form.validateFields() -> 只验证并返回当前渲染在 DOM 中的表单项
+ form.getFieldsValue(true) -> 返回整个表单的所有字段值，无论它们是否当前显示在 DOM 中 
  - 通常于 Form.Item 的 noStyle、hidden 联用，辅助处理表单数据
+ form.setFieldsValue({ name: 'xxx' }) -> 设置对应 name 的值
+ initialValues -> 只在组件初次渲染时生效, 一般设置在 Form 标签上，而不是 Form.Item -> 如果是异步获取的初始化参数，最好用 useEffect 来赋值
+ Form.useWatch(form, 'xxx') -> 监听表单某个字段的变化

**Form.useWatch 和 form.getFieldsValue区别**
1. 响应式 vs 静态
Form.useWatch（响应式）：当字段值变化时，value 会自动更新，组件重新渲染
const value = Form.useWatch('fieldName', form);

getFieldsValue（静态）：只在调用时获取当前值，不会自动触发重新渲染
const value = form.getFieldsValue(['fieldName']);

2. 使用场景
Form.useWatch 适用于：
+ 需要监听字段变化并实时响应
+ 动态显示/隐藏其他字段
+ 字段间的联动验证
+ 在函数组件中建立响应式依赖

getFieldsValue 适用于：
+ 事件处理函数中获取值（如提交时）
+ 条件判断（不需要实时监听）

**antd 还支持类似于 loash 的路径语法来绑定数据**
一般来说，Form.Item 的 name 与 后端的字段需要一一对应，这样 value 就直接展示了
但是，如果 name 对应的数据来自于其他的接口，那么可以直接用 useEffect 和 form.setFieldsValue({ xxx: xxx }) 直接给 name 对应的字段赋值，也可以不用配合 noStyle 和 hidden 来处理 form 表单数据
比如说，
```jsx
// 初始化
  useEffect(() => {
    if (dataList.length > 0) {
      const config = dataList.map((item) => ({
        [item.name]:
          item.value !== undefined && item.value !== null ? item.value : item.defaultValue,
      }))

      form.setFieldsValue({ config })
    }
  }, [dataList, form])

// 这个是在一个循环中的代码
// 使用数组索引 + 动态key访问: config[index][name]
const fieldName = ['config', index, name] // index 和 name 都是变的

// 用数据的方式绑定 name
<Form.Item
  key={name}
  {...formItemLayout}
  label={map[name]}
  name={fieldName}
  rules={[{ required: true, message: <FormattedMessage id="word.select" /> }]}
>
  <Radio.Group options={handleOptions(range)} disabled={disabled} />
</Form.Item>
```

**表单布局**
+ layout -> 控制整体布局 horizontal | vertical | inline
+ labelCol -> { span: 3, offset: 12 } 
+ wrapperCol -> { span: 3, offset: 12 } 
+ labelAlign -> left | right

span: 标签占多宽 
offest: 向左偏离多少格

**校验规则**
一般在 Form.Item 上，设置 rules
+ validator -> 自定义校验规则
+ validateTrigger -> 校验时机

注意：
如果 Form.Item 上设置了 name 属性，那么就变成了 受控组件，defaultValue 就没有用了，需要 initailValues 来设置初始值

### Form.List
将某个组件作为 表单项 遍历渲染

```jsx
<Form.List name="rules">
  {(fields, { add, remove }) => {
    return (
        <Table
          dataSource={fields}
          columns={getColumns(remove, form)}
          rowKey="key"
          pagination={false}
          style={{ width: '100%', height: '100%' }}
          scroll={{ y: '400px' }}
        />
    )
  }}
</Form.List>

// Table 中的 columns form 项
{
    title: "用户",
    dataIndex: 'user',
    width: '20%',
    key: 'user',
    align: 'center',
    render: (_, field) => {
      return (
      <Form.Item noStyle name={[field.name, 'user']}>
        <Input placeholder="请输入" />
      </Form.Item>
    )
  },
}
```
注意：fields -> 就是 xxx 的数据，在 Form.List 下的 Form.Item name值需要设置成 [field.name, 'user']，最后获取数据 -> rules: [ { user: 'xxx' } ]






