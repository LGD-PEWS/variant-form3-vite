<template>
  <el-config-provider :locale="elLocale">
    <div id="app">
      <VFormDesigner ref="vfDesignerRef" :global-dsv="globalDsv">

        <!-- <template #customToolButtons>
          <el-button type="text" @click="doTest">测试btn</el-button>
          <el-button type="text" @click="saveFormJson">保存</el-button>
        </template> -->

      </VFormDesigner>
    </div>
  </el-config-provider>
</template>

<script>
import VFormDesigner from './components/form-designer/index.vue'

import zhCNLang from 'element-plus/lib/locale/lang/zh-cn'
import enUSLang from 'element-plus/lib/locale/lang/en'

export default {
  name: 'App',
  components: {
    VFormDesigner,
  },
  data() {
    return {
      elLocaleMap: {
        'zh-CN': zhCNLang,
        'en-US': enUSLang,
      },

      //全局数据源变量
      globalDsv: {
        testApiHost: 'http://www.test.com/api',
        testPort: 8080,
      },

    }
  },
  computed: {
    elLocale() {
      let curLocale = localStorage.getItem('v_form_locale') || 'zh-CN'
      return this.elLocaleMap[curLocale]
    },

  },
  methods: {
    doTest() {
      let fieldList = this.$refs.vfDesignerRef.getFieldWidgets(null, true)
      console.log('test', fieldList)
    },
    saveFormJson() {
      let formJson = this.$refs.vfDesignerRef.getFormJson()
      console.log(formJson)
      //TODO: 将formJson提交给后端保存接口，需自行实现！！
      // ElMessage.success('表单已保存！')
    }

  }
}
</script>

<style lang="scss">
#app {
  height: 100%;
}
</style>
