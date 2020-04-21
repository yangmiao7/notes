## 写在前面
> 最近的工作，都是一些后台管理类项目，涉及到表单的使用上，有着大量的相同内容的表单，根据使用场景分为**新建表单**与**编辑表单**。在日常的搬砖过程中，对此类出现较多的场景做了一些思考。

以下将通过一个小案例结合几种常见场景一步一步进行分析并实现。

## 举个栗子
> 实现一个员工管理功能，其中包含员工列表，新增员工，修改员工信息功能。
- 员工列表
![](https://user-gold-cdn.xitu.io/2020/3/26/171127916644d102?w=1224&h=362&f=png&s=26181)
- 新增员工表单

![](https://user-gold-cdn.xitu.io/2020/3/26/171127959cfb29f5?w=716&h=674&f=png&s=22857)
- 编辑员工表单

![](https://user-gold-cdn.xitu.io/2020/3/26/1711279b3ad115c1?w=690&h=620&f=png&s=22854)

上面图中可以看出，员工的编辑表单和删除表单基本一致。

## 实现
以下涉及到的代码使用element组件，其实无论使用iview、element或其他vue组件库，思路上没有太多差别。

### 最差实现
> 没有封装，实现一个功能后，另一个功能代码直接复制。开发过程中见过不少人采用这种方式进行开发，这可能是完成功能最快的方式，同时也是带来问题最多的。

1. 不容易维护，复制的代码中如果存在bug，相当于将bug也复制了一份。后续解决bug，或者添加新功能需要花经历修改两处代码。比如：表单需要增加一项出生年月的表单项，需要在添加和编辑的代码中都进行修改。
2. 代码冗余重复率高，常见的代码质量管理的工具中都会包含有重复率一项。
3. …
总之，复制粘贴代码违反了DRY原则：系统的每一个功能都应该有唯一的实现。如果多次遇到同样的问题，就应该抽象出一个共同的解决方法，不要重复开发同样的功能。

### 封装组件，提取整个弹窗
> 将整个弹出层和表单封装成组件，新建和编辑功能直接调用。
平时看同事代码时，发现这也是同事们用的比较多的实现方式。
```vue
<template>
  <ElDialog
      :title="title"
      :visible="visible"
      @update:visible="handleVisibleChange"
  >
    <ElForm ref="EditForm" :model="form" :rules="rules">
      <ElFormItem label="姓名" prop="name">
        <ElInput v-model="form.name"/>
      </ElFormItem>
      <ElFormItem label="身份证号码" prop="cid">
        <ElInput v-model="form.cid"/>
      </ElFormItem>
      <ElFormItem label="联系地址" prop="address">
        <ElInput v-model="form.address"/>
      </ElFormItem>
    </ElForm>
    
    <template #footer>
      <ElButton @click="handleVisibleChange(false)">取消</ElButton>
      <ElButton type="primary" @click="handleSave">保存</ElButton>
    </template>
  </ElDialog>
</template>
<script>
  export default {
    name: "EditForm",
    props: {
      // 是否显示表单
      visible: {
        type: Boolean,
        default: false
      },
      // 弹窗的title
      title: String,
      // 回显数据
      model: {
        type: Object,
        default: null
      }
    },
    data() {
      return {
        form: {
          cid: '',
          name: '',
          address: ''
        },
        rules: {
          name: {required: true, message: '请输入姓名', trigger: 'blur'},
          cid: {required: true, message: '请输入身份证号码', trigger: 'blur'},
          address: {required: true, message: '请输入联系地址', trigger: 'blur'}
        },
      }
    },
    watch: {
      // 监听 编辑时回显表单
      model(employeeInfo) {
          this.form = {...employeeInfo} // 简单的浅克隆
        }
    },
    methods: {
      handleSave() {
        // 表单验证 返回数据
        this.$refs.EditForm.validate((valid) => {
          if (valid) {
            this.$emit('save', this.form)
          }
        })
      },
      handleVisibleChange(value) {
        this.$emit('update:visible', value)
      }
    }
  }
</script>
```
在父组件中，只需要通过props向表单组件传递一些必要的数据，即实现了新增和编辑功能。

编辑时可以通过传入名为model的props，表单组件中通过watch的model的变化实现表单的回显。

```
<template>
  <div>
    <EditForm
        title="新增员工"
        :visible.sync="showAddForm"
        @save="handleAddEmployee"
    />
    <EditForm
        :model="editFormData"
        title="编辑员工"
        :visible.sync="showEditForm"
        @save="handleEditEmployee"
    />
  </div>
</template>
<script>
  import EditForm from "./EditForm";
  export default {
    components: {
      EditForm
    },
    data() {
      return {
        showEditForm: false, // 是否显示编辑表单
        showAddForm: false, // 是否显示新增表单
        editFormData: {}
      }
    },
    methods: {
      // 添加一个员工
      handleAddEmployee(employeeInfo) {
        // do something
      },
      // 编辑一个员工
      handleEditEmployee(employeeInfo) {
         // do something
      }
    }
  }
</script>
```
> 现在，编辑和新建复用了同一个组件。如果需求变更：增加一个出生年月的表单项，之后只需要修改这一个文件，就完成两个功能的修改。

这种实现方式已经可以满足我们目前的需求，但是**还是会存在一些问题**：

1. **不符合单一职责原则**：现在表单组件中既封装了表单中的数据和功能，也有操作按钮，Dialog组件的一些数据及操作（如：visible状态），表单组件中通过定义title、visible等props对Dialog所需的props进行了一次中转，产生了一些额外的代码。</br> 不仅如此，假设现在需要对保存按钮也进行区分：编辑员工时，保存按钮文案改为编辑，或者在新建员工时增加暂存按钮及功能。我们就需要通过增加表单组件的props，或在组件内部进行判断来实现，这里通过增加按钮或修改文案举例，实际上通过slot插槽或其他方式也可以解决这个问题，真实的业务场景可能更为复杂。目的是想说明这些没有涉及到表单功能的修改，但是我们依然需要修改同一个组件。
2. **组件覆盖的场景比较少**，在案例中，我们的添加修改都是弹窗的形式，但是在更复杂的真实业务中，可能需要通过一个页面进行添加员工，编辑通过弹窗实现。或者需要实现批量添加员工这样的功能，因为我们的组件内部也集成了Dialog，就没办法很好的完成。


### 更进一步，剥离Dialog和按钮
> 基于对第二种方式出现问题的思考，我们可以对表单组件与Dialog，操作按钮进行剥离，进一步抽象表单组件。
```
<template>
  <ElForm ref="EditForm" :model="form" :rules="rules">
    <ElFormItem label="姓名" prop="name">
      <ElInput v-model="form.name"/>
    </ElFormItem>
    <ElFormItem label="身份证号码" prop="cid">
      <ElInput v-model="form.cid"/>
    </ElFormItem>
    <ElFormItem label="联系地址" prop="address">
      <ElInput v-model="form.address"/>
    </ElFormItem>
  </ElForm>
</template>
<script>
  export default {
    name: "EditForm",
    props: {
      // 回显数据
      model: {
        type: Object,
        default: null
      }
    },
    data() {
      return {
        form: {
          cid: '',
          name: '',
          address: ''
        },
        rules: {
          name: {required: true, message: '请输入姓名', trigger: 'blur'},
          cid: {required: true, message: '请输入身份证号码', trigger: 'blur'},
          address: {required: true, message: '请输入联系地址', trigger: 'blur'}
        },
      }
    },
    watch: {
      // 监听 编辑时回显表单
      model(employeeInfo) {
        this.form = {...employeeInfo} // 简单的浅克隆
      }
    },
    methods: {
      // 对外暴露获取数据的方法，内部进行表单的校验，父组件中通过refs调用
      getValue() {
        return new Promise((resolve, reject) => {
          this.$refs.EditForm.validate((valid) => {
            if (valid) {
              resolve({...this.form})
            } else {
              reject('表单校验没通过，可以抛出一个异常')
            }
          })
        })
      }
    }
  }
</script>
```
> 在剥离了Dialog和操作按钮后，相关的中转props的逻辑也被剥离出了组件。

由于保存按钮现在不在表单组件中了，没办法通过emit的方式对父组件暴露表单数据，所以改为在methods中注册了一个getValue方法，方法中进行了表单验证，通过返回一个Promise的方式，返回表单数据或异常。父组件在使用这个新的表单组件时，通过ref注册一个表单组件的引用，调用表单组件中的getValue方法获取组件内部的数据。

具体操作如下：
```
<template>
  <div>
    <!-- 新建时 -->
    <Dialog :visible.sync="showAddForm">
      <NewEditForm ref="EditForm"/>
      
      <template #footer>
        <ElButton @click="showAddForm = false">取消</ElButton>
        <ElButton type="primary" @click="handleAddEmployee">保存</ElButton>
      </template>
    </Dialog>
    
    <!-- 编辑时 -->
    <ElDialog :visible.sync="showEditForm">
      <NewEditForm ref="AddForm" :model="editFormData"/>
      
      <template #footer>
        <ElButton @click="showEditForm = false">取消</ElButton>
        <ElButton type="primary" @click="handleEditEmployee">保存</ElButton>
      </template>
    </ElDialog>
  </div>
</template>
<script>
  import NewEditForm from "./NewEditForm";
  export default {
    components: {
      NewEditForm
    },
    data() {
      return {
        showEditForm: false, // 是否显示编辑表单
        showAddForm: false // 是否显示新建表单
    },
    methods: {
      ...
      // 编辑一个员工
      handleEditEmployee() {
        this.$refs.NewEditForm.getValue()
          .then((employeeInfo) => {
            // 调用编辑接口
          })
          .catch((error) => {
            // 处理表单验证失败
          })
      },
      // 添加一个员工
      handleAddEmployee() {
        this.$refs.AddForm.getValue()
          .then((employeeInfo) => {
            // 调用添加接口
          })
          .catch((error) => {
            // 处理表单验证失败
          })
      }
    }
  }
</script>
```
通过这样封装，当我们需要修改Dialog或者操作按钮，直接在父组件中修改即可。表单组件粒度更小也更灵活，可以覆盖到更多的使用场景，想要实现一个添加员工页面或 **批量添加员工**，就方便很多。

## 实现批量添加员工
> 通过```v-for```循环表单组件实现批量添加员工表单功能。

![](https://user-gold-cdn.xitu.io/2020/3/25/171125ca71ef967e?w=1095&h=775&f=gif&s=239761)

代码如下：
```
<template>
  <div>
    <ElDialog title="批量添加员工" :visible="showForm">
      <div v-for="(symble, index) in employeeList" :key="symble">
        <div class="form-header">
          序号：{{index + 1}}

          <ElButton type="text" @click="handleRemoveForm(symble)">删除</ElButton>
        </div>
        <NewEditForm ref="EmployeeForm"/>
        <hr/>
      </div>

      <ElButton
          type="primary"
          style="display: block"
          @click="handleAddForm"
      >
        添加员工
      </ElButton>

      <template #footer>
        <ElButton @click="showForm = false">取消</ElButton>
        <ElButton type="primary" @click="handleSave">保存</ElButton>
      </template>
    </ElDialog>
  </div>
</template>

<script>
  import NewEditForm from "./NewEditForm";

  export default {
    components: {
      NewEditForm
    },

    data() {
      return {
        showForm: true,
        employeeList: [Symbol()] // 删除功能需要提供唯一key值才能保证严谨
      }
    },

    methods: {
      // 新增一个表单
      handleAddForm() {
        this.employeeList.push(Symbol())
      },

      // 移除表单
      handleRemoveForm(symble) {
        this.employeeList.splice(this.employeeList.indexOf(symble), 1)
      },

      // 通过refs.getValue方法配合Promise.all方法
      handleSave() {
        Promise.all(
          this.$refs.EmployeeForm.map(formRef => {
            return formRef.getValue()
          })
        )
          .then(formData => {
            // do something
          })
          .catch(error => {
            // 处理验证失败的异常
          })
      }
    }
  }
</script>
```

因为考虑到如果有删除单个员工表单功能，```v-for```循环的是一个装有```Symbol```的数组，做为每个表单唯一的```key```值。

点击保存时则通过循环调用每个表单组件中的getValue方法，将返回的```Promise```对象放入数组，最后通过```Promise.all```方法获取表单数据或处理异常。

（完。

## 最后
这是我第一篇文章。

平时工作也遇到过不少问题，也学习到了不少的优秀的代码，由于没有记录的习惯，总是会忘掉，希望自己能养成定期总结、思考的习惯💪。

