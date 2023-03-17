<template>
  <el-scrollbar class="side-scroll-bar" :style="{ height: scrollerHeight }">
    <div class="panel-container">

      <el-tabs v-model="firstTab" class="no-bottom-margin indent-left-margin">
        <el-tab-pane name="componentLib">
          <template #label>
            <span><svg-icon icon-class="el-set-up" /> {{ i18nt('designer.componentLib') }}</span>
          </template>

          <el-collapse v-model="activeNames" class="widget-collapse">
            <el-collapse-item name="1" :title="i18nt('designer.containerTitle')">
              <draggable tag="ul" :list="containers" item-key="key"
                :group="{ name: 'dragGroup', pull: 'clone', put: false }" :clone="handleContainerWidgetClone"
                ghost-class="ghost" :sort="false" :move="checkContainerMove" @end="onContainerDragEnd">
                <template #item="{ element: ctn }">
                  <li class="container-widget-item" :title="ctn.displayName" @dblclick="addContainerByDbClick(ctn)">
                    <span><svg-icon :icon-class="ctn.icon" class-name="color-svg-icon" />{{
                      i18n2t(`designer.widgetLabel.${ctn.type}`,
                        `extension.widgetLabel.${ctn.type}`) }}</span>
                  </li>
                </template>
              </draggable>
            </el-collapse-item>

            <el-collapse-item name="2" :title="i18nt('designer.basicFieldTitle')">
              <draggable tag="ul" :list="basicFields" item-key="key"
                :group="{ name: 'dragGroup', pull: 'clone', put: false }" :move="checkFieldMove"
                :clone="handleFieldWidgetClone" ghost-class="ghost" :sort="false">
                <template #item="{ element: fld }">
                  <li class="field-widget-item" :title="fld.displayName" @dblclick="addFieldByDbClick(fld)">
                    <span><svg-icon :icon-class="fld.icon" class-name="color-svg-icon" />{{
                      i18n2t(`designer.widgetLabel.${fld.type}`,
                        `extension.widgetLabel.${fld.type}`) }}</span>
                  </li>
                </template>
              </draggable>
            </el-collapse-item>

            <el-collapse-item name="3" :title="i18nt('designer.advancedFieldTitle')">
              <draggable tag="ul" :list="advancedFields" item-key="key"
                :group="{ name: 'dragGroup', pull: 'clone', put: false }" :move="checkFieldMove"
                :clone="handleFieldWidgetClone" ghost-class="ghost" :sort="false">
                <template #item="{ element: fld }">
                  <li class="field-widget-item" :title="fld.displayName" @dblclick="addFieldByDbClick(fld)">
                    <span><svg-icon :icon-class="fld.icon" class-name="color-svg-icon" />{{
                      i18n2t(`designer.widgetLabel.${fld.type}`,
                        `extension.widgetLabel.${fld.type}`) }}</span>
                  </li>
                </template>
              </draggable>
            </el-collapse-item>

            <el-collapse-item name="4" :title="i18nt('designer.customFieldTitle')">
              <draggable tag="ul" :list="customFields" item-key="key"
                :group="{ name: 'dragGroup', pull: 'clone', put: false }" :move="checkFieldMove"
                :clone="handleFieldWidgetClone" ghost-class="ghost" :sort="false">
                <template #item="{ element: fld }">
                  <li class="field-widget-item" :title="fld.displayName" @dblclick="addFieldByDbClick(fld)">
                    <span>
                      <svg-icon :icon-class="fld.icon" class-name="color-svg-icon" />{{
                        i18n2t(`designer.widgetLabel.${fld.type}`,
                          `extension.widgetLabel.${fld.type}`) }}</span>
                  </li>
                </template>
              </draggable>
            </el-collapse-item>

          </el-collapse>

        </el-tab-pane>

        <!-- 表单模板 -->
        <el-tab-pane v-if="showFormTemplates()" name="formLib" style="padding: 8px">
          <template #label>
            <span><svg-icon icon-class="el-form-template" /> {{ i18nt('designer.formLib') }}</span>
          </template>

          <template v-for="(ft, idx) in formTemplates">
            <el-card :bord-style="{ padding: '0' }" shadow="hover" class="ft-card">
              <el-popover placement="right" trigger="hover">
                <template #reference>
                  <img :src="ft.imgUrl" style="width: 200px">
                </template>
                <img :src="ft.imgUrl" style="height: 600px;width: 720px">
              </el-popover>
              <div class="bottom clear-fix">
                <span class="ft-title">#{{ idx + 1 }} {{ ft.title }}</span>
                <el-button link type="primary" class="right-button" @click="loadFormTemplate(ft.jsonUrl)">
                  {{ i18nt('designer.hint.loadFormTemplate') }}</el-button>
              </div>
            </el-card>
          </template>
        </el-tab-pane>

      </el-tabs>

    </div>
  </el-scrollbar>
</template>

<script>
import { containers as CONS, basicFields as BFS, advancedFields as AFS, customFields as CFS } from "./widgetsConfig"
import { formTemplates } from './templatesConfig'
import { addWindowResizeHandler, generateId } from "@/utils/util"
import i18n from "@/utils/i18n"
import axios from 'axios'
import SvgIcon from "@/components/svg-icon/index";

// import ftImg1 from '@/assets/ft-images/t1.png'
// import ftImg2 from '@/assets/ft-images/t2.png'
// import ftImg3 from '@/assets/ft-images/t3.png'
// import ftImg4 from '@/assets/ft-images/t4.png'
// import ftImg5 from '@/assets/ft-images/t5.png'
// import ftImg6 from '@/assets/ft-images/t6.png'
// import ftImg7 from '@/assets/ft-images/t7.png'
// import ftImg8 from '@/assets/ft-images/t8.png'

export default {
  name: "FieldPanel",
  mixins: [i18n],
  components: {
    SvgIcon
  },
  props: {
    designer: Object,
  },
  inject: ['getBannedWidgets', 'getDesignerConfig'],
  data() {
    return {
      designerConfig: this.getDesignerConfig(),

      firstTab: 'componentLib',

      scrollerHeight: 0,

      activeNames: ['1', '2', '3', '4'],

      containers: [],
      basicFields: [],
      advancedFields: [],
      customFields: [],

      formTemplates: formTemplates,
      // ftImages: [
      //   {imgUrl: ftImg1},
      //   {imgUrl: ftImg2},
      //   {imgUrl: ftImg3},
      //   {imgUrl: ftImg4},
      //   {imgUrl: ftImg5},
      //   {imgUrl: ftImg6},
      //   {imgUrl: ftImg7},
      //   {imgUrl: ftImg8},
      // ]
    }
  },
  computed: {
    //
  },
  created() {
    this.loadWidgets()
  },
  mounted() {
    //this.loadWidgets()

    // this.scrollerHeight = window.innerHeight - 56 + 'px'
    this.scrollerHeight = window.innerHeight - 0 + 'px'
    addWindowResizeHandler(() => {
      this.$nextTick(() => {
        this.scrollerHeight = window.innerHeight - 0 + 'px'
        //console.log(this.scrollerHeight)
      })
    })
  },
  methods: {
    isBanned(wName) {
      return this.getBannedWidgets().indexOf(wName) > -1
    },

    showFormTemplates() {
      if (this.designerConfig['formTemplates'] === undefined) {
        return true
      }

      return !!this.designerConfig['formTemplates']
    },

    loadWidgets() {
      this.containers = CONS.map(con => {
        return {
          key: generateId(),
          ...con,
          displayName: this.i18n2t(`designer.widgetLabel.${con.type}`, `extension.widgetLabel.${con.type}`)
        }
      }).filter(con => {
        return !con.internal && !this.isBanned(con.type)
      })

      this.basicFields = BFS.map(fld => {
        return {
          key: generateId(),
          ...fld,
          displayName: this.i18n2t(`designer.widgetLabel.${fld.type}`, `extension.widgetLabel.${fld.type}`)
        }
      }).filter(fld => {
        return !this.isBanned(fld.type)
      })

      this.advancedFields = AFS.map(fld => {
        return {
          key: generateId(),
          ...fld,
          displayName: this.i18n2t(`designer.widgetLabel.${fld.type}`, `extension.widgetLabel.${fld.type}`)
        }
      }).filter(fld => {
        return !this.isBanned(fld.type)
      })

      this.customFields = CFS.map(fld => {
        return {
          key: generateId(),
          ...fld,
          displayName: this.i18n2t(`designer.widgetLabel.${fld.type}`, `extension.widgetLabel.${fld.type}`)
        }
      }).filter(fld => {
        return !this.isBanned(fld.type)
      })
    },

    handleContainerWidgetClone(origin) {
      return this.designer.copyNewContainerWidget(origin)
    },

    handleFieldWidgetClone(origin) {
      return this.designer.copyNewFieldWidget(origin)
    },

    checkContainerMove(evt) {
      return this.designer.checkWidgetMove(evt)
    },

    checkFieldMove(evt) {
      return this.designer.checkFieldMove(evt)
    },

    onContainerDragEnd(evt) {
      //console.log('Drag end of container: ')
      //console.log(evt)
    },

    addContainerByDbClick(container) {
      this.designer.addContainerByDbClick(container)
    },

    addFieldByDbClick(widget) {
      this.designer.addFieldByDbClick(widget)
    },

    loadFormTemplate(jsonUrl) {
      this.$confirm(this.i18nt('designer.hint.loadFormTemplateHint'), this.i18nt('render.hint.prompt'), {
        confirmButtonText: this.i18nt('render.hint.confirm'),
        cancelButtonText: this.i18nt('render.hint.cancel')
      }).then(() => {
        axios.get(jsonUrl).then(res => {
          console.log("获取表单模板", res)
          let modifiedFlag = false
          if (1) {
            modifiedFlag = this.designer.loadFormJson({ "widgetList": [{ "key": 57237, "type": "input", "icon": "text-field", "formItemFlag": true, "options": { "name": "input", "label": "input", "labelAlign": "", "type": "text", "defaultValue": "", "placeholder": "123", "columnWidth": "200px", "size": "", "labelWidth": null, "labelHidden": false, "readonly": false, "disabled": false, "hidden": false, "clearable": true, "showPassword": false, "required": true, "requiredHint": "", "validation": "", "validationHint": "", "customClass": [], "labelIconClass": null, "labelIconPosition": "rear", "labelTooltip": null, "minLength": null, "maxLength": null, "showWordLimit": false, "prefixIcon": "", "suffixIcon": "", "appendButton": false, "appendButtonDisabled": false, "buttonIcon": "custom-search", "onCreated": "", "onMounted": "", "onInput": "", "onChange": "", "onFocus": "", "onBlur": "", "onValidate": "", "onAppendButtonClick": "" }, "id": "input105382" }, { "key": 57237, "type": "input", "icon": "text-field", "formItemFlag": true, "options": { "name": "input", "label": "input", "labelAlign": "", "type": "text", "defaultValue": "", "placeholder": "", "columnWidth": "200px", "size": "", "labelWidth": null, "labelHidden": false, "readonly": false, "disabled": false, "hidden": false, "clearable": true, "showPassword": false, "required": false, "requiredHint": "", "validation": "", "validationHint": "", "customClass": "", "labelIconClass": null, "labelIconPosition": "rear", "labelTooltip": null, "minLength": null, "maxLength": null, "showWordLimit": false, "prefixIcon": "", "suffixIcon": "", "appendButton": false, "appendButtonDisabled": false, "buttonIcon": "custom-search", "onCreated": "", "onMounted": "", "onInput": "", "onChange": "", "onFocus": "", "onBlur": "", "onValidate": "", "onAppendButtonClick": "" }, "id": "input42612" }, { "key": 76486, "type": "textarea", "icon": "textarea-field", "formItemFlag": true, "options": { "name": "textarea", "label": "textarea", "labelAlign": "", "rows": 3, "defaultValue": "", "placeholder": "", "columnWidth": "200px", "size": "", "labelWidth": null, "labelHidden": false, "readonly": false, "disabled": true, "hidden": false, "required": false, "requiredHint": "", "validation": "", "validationHint": "", "customClass": "", "labelIconClass": null, "labelIconPosition": "rear", "labelTooltip": null, "minLength": null, "maxLength": null, "showWordLimit": false, "onCreated": "", "onMounted": "", "onInput": "", "onChange": "", "onFocus": "", "onBlur": "", "onValidate": "" }, "id": "textarea41187" }, { "key": 16977, "type": "radio", "icon": "radio-field", "formItemFlag": true, "options": { "name": "radio", "label": "radio", "labelAlign": "", "defaultValue": 2, "columnWidth": "200px", "size": "", "displayStyle": "inline", "buttonStyle": false, "border": false, "labelWidth": null, "labelHidden": false, "disabled": false, "hidden": false, "optionItems": [{ "label": "radio 1", "value": 1 }, { "label": "radio 2", "value": 2 }, { "label": "radio 3", "value": 3 }], "required": false, "requiredHint": "", "validation": "", "validationHint": "", "customClass": "", "labelIconClass": null, "labelIconPosition": "rear", "labelTooltip": null, "onCreated": "", "onMounted": "", "onChange": "", "onValidate": "" }, "id": "radio46488" }, { "key": 103308, "type": "checkbox", "icon": "checkbox-field", "formItemFlag": true, "options": { "name": "checkbox", "label": "checkbox", "labelAlign": "", "defaultValue": [], "columnWidth": "200px", "size": "", "displayStyle": "inline", "buttonStyle": false, "border": false, "labelWidth": null, "labelHidden": false, "disabled": false, "hidden": false, "optionItems": [{ "label": "check 1", "value": 1 }, { "label": "check 2", "value": 2 }, { "label": "check 3", "value": 3 }], "required": false, "requiredHint": "", "validation": "", "validationHint": "", "customClass": "", "labelIconClass": null, "labelIconPosition": "rear", "labelTooltip": null, "onCreated": "", "onMounted": "", "onChange": "", "onValidate": "" }, "id": "checkbox52024" }], "formConfig": { "modelName": "formData", "refName": "vForm", "rulesName": "rules", "labelWidth": 80, "labelPosition": "left", "size": "", "labelAlign": "label-left-align", "cssCode": "", "customClass": [], "functions": "", "layoutType": "PC", "jsonVersion": 3, "onFormCreated": "", "onFormMounted": "", "onFormDataChange": "" } })

            // if (typeof res.data === 'string') {
            // console.log("1")
            // 用于获取
            // modifiedFlag = this.designer.loadFormJson(JSON.parse(res.data))
          } else if (res.data.constructor === Object) {
            console.log("2")

            modifiedFlag = this.designer.loadFormJson(res.data)
          }
          if (modifiedFlag) {
            console.log("3")

            this.designer.emitHistoryChange()
          }

          this.$message.success(this.i18nt('designer.hint.loadFormTemplateSuccess'))
        }).catch(error => {
          this.$message.error(this.i18nt('designer.hint.loadFormTemplateFailed') + ':' + error)
        })
      }).catch(error => {
        console.error(error)
      })
    }

  }

}
</script>

<style lang="scss" scoped>
.color-svg-icon {
  color: $--color-primary;
}

.side-scroll-bar {
  :deep(.el-scrollbar__wrap) {
    overflow-x: hidden;
  }
}

div.panel-container {
  padding-bottom: 10px;
}

.no-bottom-margin :deep(.el-tabs__header) {
  margin-bottom: 0;
}

.indent-left-margin {
  :deep(.el-tabs__nav) {
    margin-left: 20px;
  }
}

.el-collapse-item :deep(ul)>li {
  list-style: none;
}

.widget-collapse {
  border-top-width: 0;

  :deep(.el-collapse-item__header) {
    margin-left: 8px;
    font-style: italic;
    font-weight: bold;
  }

  :deep(.el-collapse-item__content) {
    padding-bottom: 6px;

    ul {
      padding-left: 10px;
      /* 重置IE11默认样式 */
      margin: 0;
      /* 重置IE11默认样式 */
      margin-block-start: 0;
      margin-block-end: 0.25em;
      padding-inline-start: 10px;

      &:after {
        content: "";
        display: block;
        clear: both;
      }

      .container-widget-item,
      .field-widget-item {
        display: inline-block;
        height: 28px;
        line-height: 28px;
        width: 115px;
        float: left;
        margin: 2px 6px 6px 0;
        cursor: move;
        white-space: nowrap;
        text-overflow: ellipsis;
        overflow: hidden;
        background: #f1f2f3;
      }

      .container-widget-item:hover,
      .field-widget-item:hover {
        background: #EBEEF5;
        outline: 1px solid $--color-primary;
      }

      .drag-handler {
        position: absolute;
        top: 0;
        left: 160px;
        background-color: #dddddd;
        border-radius: 5px;
        padding-right: 5px;
        font-size: 11px;
        color: #666666;
      }
    }
  }
}

.el-card.ft-card {
  border: 1px solid #8896B3;
}

.ft-card {
  margin-bottom: 10px;

  .bottom {
    margin-top: 10px;
    line-height: 12px;
  }

  /*
    .image-zoom {
      height: 500px;
      width: 620px
    }
    */

  .ft-title {
    font-size: 13px;
    font-weight: bold;
  }

  .right-button {
    padding: 0;
    float: right;
  }

  .clear-fix:before,
  .clear-fix:after {
    display: table;
    content: "";
  }

  .clear-fix:after {
    clear: both;
  }

}
</style>
