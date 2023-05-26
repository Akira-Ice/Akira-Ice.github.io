---
title: element-plus 表单自定义验证
categories:
  - [BugRoad]
tags: 
  - BugRoad
---

# element-plus 表单自定义验证

类似我们需要验证只由 `数字和英文字母组成` 的表单：

## validator

要验证英文和数字的组合类型，可以使用 Element Plus 表单验证中的自定义验证函数来实现。自定义验证函数可以接收一个参数，即当前验证字段的值，然后根据需求进行验证并返回一个布尔值表示验证结果。

以下是一个示例，演示如何验证英文和数字的组合类型：

```html
<template>
  <el-form :model="form" :rules="rules" ref="form" label-width="80px">
    <el-form-item label="英文+数字" prop="alphanumeric">
      <el-input v-model="form.alphanumeric"></el-input>
    </el-form-item>
    <el-form-item>
      <el-button type="primary" @click="submitForm">提交</el-button>
    </el-form-item>
  </el-form>
</template>

<script>
export default {
  data() {
    return {
      form: {
        alphanumeric: ''
      },
      rules: {
        alphanumeric: [
          { required: true, message: '请输入英文和数字的组合类型', trigger: 'blur' },
          { validator: this.validateAlphanumeric, message: '请输入英文和数字的组合类型', trigger: 'blur' }
        ]
      }
    };
  },
  methods: {
    validateAlphanumeric(rule, value, callback) {
      const alphanumericRegex = /^[a-zA-Z0-9]+$/;
      if (value && !alphanumericRegex.test(value)) {
        callback(new Error('请输入英文和数字的组合类型'));
      } else {
        callback();
      }
    },
    submitForm() {
      this.$refs.form.validate(valid => {
        if (valid) {
          // 表单验证通过
          console.log('验证通过');
        } else {
          // 表单验证失败
          console.log('验证失败');
          return false;
        }
      });
    }
  }
};
</script>
```

在这个示例中，定义了一个自定义验证函数 `validateAlphanumeric`，用于验证英文和数字的组合类型。在验证函数中，使用了一个正则表达式 `/^[a-zA-Z0-9]+$/` 来匹配英文和数字的组合类型，如果验证失败则返回一个错误信息，否则返回空。

在表单验证规则中，使用了 `validator` 属性来指定自定义验证函数，并将验证失败时的错误信息设置为“请输入英文和数字的组合类型”。

需要注意的是，这个示例中的正则表达式只能验证英文和数字的组合类型，如果需要验证其他类型的组合，需要根据实际情况进行调整。同时，也可以根据需求在自定义验证函数中添加其他的验证逻辑，例如最小长度、最大长度等等。

## pattern

除了使用自定义验证函数之外，还可以使用正则表达式来验证英文和数字的组合类型。使用正则表达式可以更加灵活地控制验证规则，但需要熟悉正则表达式的语法和用法。

以下是一个示例，演示如何使用正则表达式来验证英文和数字的组合类型：

```html
<template>
  <el-form :model="form" :rules="rules" ref="form" label-width="80px">
    <el-form-item label="英文+数字" prop="alphanumeric">
      <el-input v-model="form.alphanumeric"></el-input>
    </el-form-item>
    <el-form-item>
      <el-button type="primary" @click="submitForm">提交</el-button>
    </el-form-item>
  </el-form>
</template>

<script>
export default {
  data() {
    return {
      form: {
        alphanumeric: ''
      },
      rules: {
        alphanumeric: [
          { required: true, message: '请输入英文和数字的组合类型', trigger: 'blur' },
          { pattern: /^[a-zA-Z0-9]+$/, message: '请输入英文和数字的组合类型', trigger: 'blur' }
        ]
      }
    };
  },
  methods: {
    submitForm() {
      this.$refs.form.validate(valid => {
        if (valid) {
          // 表单验证通过
          console.log('验证通过');
        } else {
          // 表单验证失败
          console.log('验证失败');
          return false;
        }
      });
    }
  }
};
</script>
```

在这个示例中，使用了正则表达式 `/^[a-zA-Z0-9]+$/` 来验证英文和数字的组合类型。在表单验证规则中，使用了 `pattern` 属性来指定正则表达式，并将验证失败时的错误信息设置为“请输入英文和数字的组合类型”。

需要注意的是，正则表达式中的 `/^` 表示匹配字符串的开头，`[a-zA-Z0-9]` 表示匹配英文和数字，`+` 表示匹配前面的字符至少一次，`$` 表示匹配字符串的结尾。因此，这个正则表达式可以确保输入的字符串仅包含英文和数字，并且不为空。

使用正则表达式的优点是可以更加灵活地控制验证规则，可以根据实际情况进行调整。但需要注意正则表达式的语法和用法，避免出现错误。
