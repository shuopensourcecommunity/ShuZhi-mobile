<template>
  <div>
    <q-card>
      <q-card-title>
        {{room.name}}
      </q-card-title>
      <q-list>
        <q-item>
          <q-item-side>
            <q-item-tile color="primary" icon="bookmark" />
          </q-item-side>
          <q-item-main>
            <q-item-tile label>教室类别</q-item-tile>
            <q-item-tile sublabel>{{room.type}}</q-item-tile>
          </q-item-main>
        </q-item>
        <q-item>
          <q-item-side>
            <q-item-tile color="primary" icon="event seat" />
          </q-item-side>
          <q-item-main>
            <q-item-tile label>教室容量</q-item-tile>
            <q-item-tile sublabel>{{room.cap}}</q-item-tile>
          </q-item-main>
        </q-item>
        <q-btn color="primary" class="full-width" @click="open()">
          预约
        </q-btn>
      </q-list>
    </q-card>
    <q-modal minimized ref="basicModal">
      <q-card flat>
        <q-card-title>请填写信息完成预约
          <span slot="subtitle" v-if="roomaviliable === false ">无法预约请更换日期或时间段</span>
        </q-card-title>
        <q-card-main>
          <m-input v-model="name" stack-label="姓名" />
          <q-input v-model="ID" type="number" stack-label="学号" />
          <q-input v-model="phone" type="number" stack-label="联系电话" />
          <!-- <q-input v-model="actname" stack-label="活动名称" /> -->
          <m-input v-model="actname" stack-label="活动名称" />
          <q-select v-model="date" stack-label="预约日期" :options="dateAvaliable" />
          <q-select v-model="time" class="full-width" stack-label="预约时间" :options="timeAvaliable" />
        </q-card-main>
        <q-card-actions align="around">
          <q-btn @click.native="$refs.basicModal.close()" label="Close" style="width:45%;" color="primary">取消</q-btn>
          <q-btn v-if="roomaviliable === true " @click.native="register()" label="Close" style="width:45%;" color="primary">预约</q-btn>
          <q-btn v-else :disable="!progress" color="negative">无法预约</q-btn>
        </q-card-actions>
      </q-card>
    </q-modal>
  </div>
</template>

<script>
import MInput from '@/MInput.vue'
import { Loading, Dialog, Events } from 'quasar'
const today = new Date()
export default {
  props: {
    room: Object
  },
  components: {
    MInput
  },
  data() {
    return {
      today,
      index: 0,
      modal: false,
      name: '',
      ID: '',
      phone: '',
      date: '',
      actname: ' ',
      roomaviliable: true,
      time: null,
      totalAvaliable: {}
    }
  },
  computed: {
    timeAvaliable: function() {
      if (this.totalAvaliable[this.date] === undefined) {
        return []
      } else {
        return this.totalAvaliable[this.date].filter(function(item) {
          return item.value.avaliable
        })
      }
    },
    dateAvaliable: function() {
      let avaliable = []
      for (let date in this.totalAvaliable) {
        avaliable.push({
          label: date,
          value: date
        })
      }
      return avaliable
    }
  },
  methods: {
    open() {
      if (!this.$user.login) {
        Events.$emit('shuzhi:login')
        return
      }
      this.$refs.basicModal.open()
      this.$http
        .get('/api/ChangDXX/ChangDXX/GetChangDSJ', {
          params: {
            id: this.room.ID
          }
        })
        .then(response => {
          console.log(response)
          let data = response.data.data.changdxx
          for (let i = 0; i < data.length; i++) {
            let info = {}
            info.label = data[i].ShiJD
            info.value = {
              date: data[i].RiQi,
              time: data[i].ShiJD,
              avaliable: data[i].ShiJZT
            }
            if (this.totalAvaliable[data[i].RiQi] === undefined) {
              this.$set(this.totalAvaliable, data[i].RiQi, [info])
            } else {
              this.totalAvaliable[data[i].RiQi].push(info)
            }
          }
          console.log(this.totalAvaliable)
        })
    },
    register() {
      Loading.show()
      this.$http
        .post('/api/ChangDXX/ShiNCDYYXX/CreateShiNCDYYXX', {
          ZuZXXId: '2',
          ChangDXXId: this.room.ID,
          YuYRQ: this.date,
          YuYSJD: this.time.time,
          XueHao: this.ID,
          XingMing: this.name,
          LianXDH: this.phone,
          HuoDMC: this.actname,
          HuoDLX: '1',
          HuoDJJ: '测试5',
          HuoDRS: '80'
        })
        .then(response => {
          Loading.hide()
          Dialog.create({
            title: '提示',
            message: response.data.message,
            buttons: [
              {
                label: '确定',
                handler: () => {
                  this.$refs.basicModal.close()
                }
              }
            ]
          })
        })
    }
  }
}
</script>

<style>

</style>