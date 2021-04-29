<template>
  <page-view>
    <a-row>
      <a-col :span="24">
        <a-card :bordered="false" :bodyStyle="{ padding: '16px' }">
          <a-card :bordered="false" class="environment-info" :bodyStyle="{ padding: '16px' }">
            <template slot="title">
              环境信息
              <a href="javascript:void(0);" @click="handleCopyEnvironments">
                <a-icon type="copy" />
              </a>
            </template>
            <a-popover slot="extra" placement="left" :title="isLatest ? '当前为最新版本' : '有新版本'">
              <template slot="content">
                <p>{{ versionMessage }}</p>
                <a-button type="dashed" @click="handleShowVersionContent">查看详情</a-button>
              </template>
              <a-button
                :loading="checking"
                type="dashed"
                shape="circle"
                :icon="isLatest ? 'check-circle' : 'exclamation-circle'"
              ></a-button>
            </a-popover>

            <ul class="p-0 m-0 list-none">
              <li>版本：{{ environments.version }}</li>
              <li>数据库：{{ environments.database }}</li>
              <li>运行模式：{{ environments.mode }}</li>
              <li>启动时间：{{ environments.startTime | moment }}</li>
            </ul>
          </a-card>
        </a-card>
      </a-col>

      <a-col :span="24"> </a-col>
    </a-row>

    <a-modal
      :title="versionContentModalTitle"
      :visible="versionContentVisible"
      ok-text="查看更多"
      @cancel="versionContentVisible = false"
      @ok="handleOpenVersionUrl"
      :width="620"
    >
      <div v-html="versionContent"></div>
    </a-modal>
  </page-view>
</template>

<script>
import adminApi from '@/api/admin'
import axios from 'axios'
import marked from 'marked'
import { PageView } from '@/layouts'
export default {
  components: {
    PageView
  },
  data() {
    return {
      environments: {},
      checking: false,
      isLatest: false,
      latestData: {},
      versionContentVisible: false
    }
  },
  computed: {
    versionMessage() {
      return `当前版本：${this.environments.version}，${
        this.isLatest ? '已经是最新版本。' : `新版本：${this.latestData.name}，你可以点击下方按钮查看详情。`
      }`
    },
    versionContent() {
      if (this.latestData && this.latestData.body) {
        return marked(this.latestData.body)
      } else {
        return '暂无内容'
      }
    },
    versionContentModalTitle() {
      return `${this.latestData.name} 更新内容`
    }
  },
  created() {
    this.getEnvironments()
  },
  methods: {
    async getEnvironments() {
      await adminApi.environments().then(response => {
        this.environments = response.data.data
      })
      // this.checkServerUpdate()
    },
    handleCopyEnvironments() {
      const text = `版本：${this.environments.version}
数据库：${this.environments.database}
运行模式：${this.environments.mode}
User Agent：${navigator.userAgent}`
      this.$copyText(text)
        .then(message => {
          this.$log.debug('copy', message)
          this.$message.success('复制成功！')
        })
        .catch(err => {
          this.$log.debug('copy.err', err)
          this.$message.error('复制失败！')
        })
    },
    checkServerUpdate() {
      const _this = this
      _this.checking = true
      axios
        .get('https://api.github.com/repos/halo-dev/halo/releases/latest')
        .then(response => {
          const data = response.data
          _this.latestData = data
          if (data.draft || data.prerelease) {
            return
          }
          const current = _this.calculateIntValue(_this.environments.version)
          const latest = _this.calculateIntValue(data.name)
          if (current >= latest) {
            _this.isLatest = true
            return
          }
          const title = '新版本提醒'
          const content = '检测到 Halo 新版本：' + data.name + '，点击下方按钮查看最新版本。'
          _this.$notification.open({
            message: title,
            description: content,
            icon: <a-icon type="smile" style="color: #108ee9" />,
            btn: h => {
              return h(
                'a-button',
                {
                  props: {
                    type: 'primary',
                    size: 'small'
                  },
                  on: {
                    click: () => this.handleShowVersionContent()
                  }
                },
                '去看看'
              )
            }
          })
        })
        .catch(function(error) {
          this.$log.error('Check update fail', error)
        })
        .finally(() => {
          setTimeout(() => {
            this.checking = false
          }, 200)
        })
    },
    handleShowVersionContent() {
      this.versionContentVisible = true
    },
    handleOpenVersionUrl() {
      window.open(this.latestData.html_url, '_blank')
    },
    calculateIntValue(version) {
      version = version.replace(/v/g, '')
      const ss = version.split('.')
      if (ss == null || ss.length !== 3) {
        return -1
      }
      const major = parseInt(ss[0])
      const minor = parseInt(ss[1])
      const micro = parseInt(ss[2])
      if (isNaN(major) || isNaN(minor) || isNaN(micro)) {
        return -1
      }
      return major * 1000000 + minor * 1000 + micro
    }
  }
}
</script>
