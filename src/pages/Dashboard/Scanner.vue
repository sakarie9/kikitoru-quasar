<template>
  <div>
    <div class="row q-ma-sm">
      <div class="col-xs-6 col-sm-4 row q-pa-sm">
        <q-btn
          class="col"
          color="teal"
          label="扫描本地音声库"
          :disable="state === 'running'"
          @click="performScan()"
        />
      </div>

      <div class="col-xs-6 col-sm-4 row q-pa-sm">
        <q-btn
          class="col"
          color="primary"
          label="刷新音声库信息"
          :disable="state === 'running'"
          @click="performUpdate()"
        />
      </div>

      <div class="col-xs-12 col-sm-4 row q-pa-sm">
        <q-btn
          class="col"
          color="negative"
          label="终止扫描进程"
          :disable="state !== 'running'"
          @click="killScanProceess()"
        />
      </div>
    </div>

    <q-card v-show="state" class="q-ma-md">
      <div v-for="item in mainLogs" :key="item">
        <h6>{{ item }}</h6>
      </div>
      <div v-for="item in results" :key="item">
        <p style="margin-bottom: 12px;">{{ item }}</p>
      </div>
    </q-card>
  </div>
</template>

<script>
import NotifyMixin from '../../mixins/Notification.js'

export default {
  name: 'Scanner',

  mixins: [NotifyMixin],

  data () {
    return {
      tab: 'tasks',
      state: null, // ['running', 'finished', 'error']
      loggedIn: false,
      tasks: [], // 正在处理中的并行任务
      failedTasks: [], // 处理失败的任务
      mainLogs: [],
      results: [],
      timer: null
    }
  },

  methods: {
    // performScan () {
      // this.tasks = []
      // this.failedTasks = []
      // this.mainLogs = []
      // this.results = []
      // this.state = 'running'
      // this.$socket.emit('PERFORM_SCAN')
    // },
    performScan() {
      this.tasks = []
      this.failedTasks = []
      this.mainLogs = []
      this.results = []
      this.state = 'running'
      this.$axios.get('/api/scan')
        .catch(error => {
          this.showErrNotif(error.message || error)
        });
      this.setTimer()
    },

    setTimer() {
      this.fetchLogs();
      this.timer = setInterval(this.fetchLogs, 1000); // 每5秒获取一次API数据
    },

    fetchLogs() {
      this.$axios.get('/api/logs')
      .then(response => {
        this.mainLogs = response.data.main_log;
        this.results = response.data.details;
        this.state = response.data.state
        if (this.state === 'finished') {
          clearInterval(this.timer);
        }
        })
        .catch(error => {
          this.showErrNotif(error.message || error)
        });
    },

    // performUpdate () {
    //   this.tasks = []
    //   this.failedTasks = []
    //   this.mainLogs = []
    //   this.results = []
    //   this.state = 'running'
    //   this.$socket.emit('PERFORM_UPDATE')
    // },

    // killScanProceess () {
    //   this.$socket.emit('KILL_SCAN_PROCESS')
    // },
  },

  beforeDestroy() {
    clearInterval(this.timer);
  }
}
</script>
