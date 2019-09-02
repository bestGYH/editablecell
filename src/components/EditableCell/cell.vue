/**
* @Description:
* @Author: yanhui.gao
* @Date: 2019/7/24  14:45
*/
<template>
  <div class="editable-cell">
    <div v-if="editable" :class="['editable-cell-input-wrapper',singleEdit ? 'single-edit' : '']">
      <a-form :form="form">
        <a-form-item :colon="false" class="no-validate">
          <a-input v-if="showFormItem('input')" v-decorator="decorator" :placeholder="placeholder" @change="handleChange"  autocomplete="off" />
          <a-input-number :min="min" :max="max" v-if="showFormItem('number')" v-decorator="decorator" :placeholder="placeholder"  />
          <a-switch v-if="showFormItem('switch')" v-decorator="decorator" :placeholder="placeholder" @change="handleChange" :value="value"/>
          <a-date-picker :showTime="showTime" :format="format" v-if="showFormItem('datePicker')" v-decorator="decorator" :placeholder="placeholder" />
          <a-range-picker :showTime="showTime" :format="format" v-if="showFormItem('rangePicker')" v-decorator="decorator" :placeholder="placeholder" />
          <a-month-picker :showTime="showTime" :format="format" v-if="showFormItem('monthPicker')" v-decorator="decorator" :placeholder="placeholder" />
          <a-select v-if="showFormItem('select')" v-decorator="decorator" :placeholder="placeholder" @change="handleChange" :mode="selectMode" :labelInValue='selectLabelInValue'>
            <slot/>
          </a-select>
          <a-auto-complete
                  v-if="showFormItem('autoComplete')" v-decorator="decorator" :placeholder="placeholder"   @search="handleSearch" optionLabelProp="text" :disabled="notEdit"
          >
            <template slot="dataSource">
              <slot/>
            </template>
          </a-auto-complete>
        </a-form-item>
      </a-form>
      <a-icon v-if="singleEdit" type="check" class="editable-cell-icon-check" @click="check(true)" />
    </div>
    <div v-else :class="['editable-cell-text-wrapper',singleEdit ? 'single-edit' : '']">
      {{ getText || ' ' }}
      <a-icon v-if="singleEdit" type="edit" class="editable-cell-icon" @click="edit" />
    </div>
  </div>
</template>
<script>
  export default {
    data() {
      return {
        form: this.$form.createForm(this),
        editable: this.defaultEditable,
      };
    },
    props: {
      // 非编辑模式下，显示文本
      text: {
        type: [Number, String, Array, Object],
      },
      // 编辑模式下组件值
      value: {
        message: { type: [Number, String, Array, Object], default: '' },
      },
      // 表单组件类型
      formType: {
        validator(value) {
          return ['input', 'number', 'switch', 'datePicker', 'rangePicker', 'select', 'monthPicker', 'autoComplete'].indexOf(value) !== -1;
        },
        default: 'input',
      },
      // 表单校验
      decoratorOptions: {
        type: [Array, Object],
      },
      min: { type: Number, default: -Infinity },
      max: { type: Number, default: Infinity },
      // 表单默认提示
      placeholder: { type: String, default: '请输入' },
      // 支持单元格单个编辑
      singleEdit: { type: Boolean, default: false },
      // 默认编辑模式
      defaultEditable: { type: Boolean, default: false },
      // 是否自动保存
      autoSave: { type: Boolean, default: false },
      // 日期组件设置
      format: { type: String, default: 'YYYY-MM-DD' },
      showTime: { type: [Boolean, Object], default: false },
      // 是否可编辑
      notEdit: { type: Boolean, default: false },
      // 选择框是否多选
      selectMode: { type: String, default: undefined },
      // 多选选择框属性
      selectLabelInValue: { type: Boolean, default: false },
      // 多选选择框默认值
      selectDefaultValue: { type: Array },
    },
    computed: {
      // 格式化文本
      getText() {
        if (typeof this.text === 'function') {
          return this.text();
        }
        return this.text;
      },
      // 根据属性，合成表单验证装饰器
      decorator: {
        get() {
          if (this.decoratorOptions) {
            const itemDecorator = ['formName', {
              initialValue: this.value,
              ...this.decoratorOptions,
            }];
            return itemDecorator;
          }
          const itemDecorator = ['formName', {
            initialValue: this.value,
          }];
          return itemDecorator;
        },
        set(value) {
          this.decorator[1].initialValue = value; // 最后修改了msg
        },

      },
    },
    watch: {
      // 监视父页面是否可编辑属性，及时响应控件内切换
      defaultEditable(value) {
        this.editable = value;
      },
    },
    methods: {
      // 判断表单是否
      showFormItem(formType) {
        return this.formType === formType;
      },
      // 表单change事件
      handleChange(value) {
        if (this.autoSave) {
          if (this.formType === 'input') {
            this.$emit('valueChange', value.target.value);
          } else if (this.formType === 'select') {
            this.$emit('valueChange', value);
          }
        }
        // this.check(false);
      },
      // auto-complete 搜索
      handleSearch(value) {
        this.$emit('autoCompleteSearch', value);
      },
      // 表单校验
      validate() {
        const result = {
          success: false,
          value: '',
        };
        this.form.validateFields((err, values) => {
          if (!err) {
            if (!this.editable) {
              result.value = this.getText;
            } else {
              result.value = values.formName;
            }
            result.success = true;
          }
        });
        return result;
      },
      // 表单赋值
      setValue(value) {
        this.form.setFieldsValue({
          formName: value,
        });
      },
      // 点击"对号"图标，校验通过后触发通知父组件
      check(flag) {
        this.form.validateFields((err, values) => {
          if (!err) {
            if (flag) {
              // 切换到只读模式
              this.editable = false;
            }
            this.$emit('valueChange', values.formName);
          }
        });
      },
      // 切换到编辑模式
      edit() {
        this.editable = true;
        this.decorator = this.value;
      },
    },
    created() {
    },
  };
</script>

<style scoped>
  .editable-cell {
    position: relative;
    margin-top: -9px;
    margin-bottom: -9px;
  }

  .editable-cell:hover .editable-cell-icon {
    display: inline-block;
  }
  .editable-cell-input-wrapper.single-edit,
  .editable-cell-text-wrapper.single-edit {
    padding-right: 24px;
  }

  .editable-cell-icon,
  .editable-cell-icon-check {
    position: absolute;
    right: 0;
    width: 20px;
    cursor: pointer;
  }
  .editable-cell-icon {
    top: 3px;
    line-height: 18px;
    display: none;
  }
  .editable-cell-icon-check {
    top: 12px;
  }

  .no-validate {
    margin-bottom: 0;
  }
</style>
