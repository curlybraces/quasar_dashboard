<template>
  <q-page>
    <div class="q-pa-md">
      <!-- <div class="row justify-center q-pa-sm">
        <q-btn-toggle
          glossy
          no-caps
          v-model="slide"
          :options="[
            { label: 'Overview', value: 'overview' },
            { label: 'Rack', value: 'rack' },
            { label: 'Cage', value: 'cage' }
          ]"
        />
      </div> -->
      <q-carousel
        v-model="slide"
        transition-prev="slide-right"
        transition-next="slide-left"
        animated
        :fullscreen.sync="fullscreen"
        class="rounded-borders"
        style="height:auto; "
      >
        <!-- Slide 1: Rack -->
        <q-carousel-slide name="overview" class="column no-wrap" >
          <div class="row justify-center q-pa-sm">
            <div class="col-12">
              <q-card
                style="width:auto-filled; height:auto; float:center;  text-align:center;"
              >
                <q-card-section style="width:auto-fill; height:auto;">
                  <!-- RackNAME + CageID + Notify -->
                  <div></div>
                  <!-- <div>
                    <q-btn
                      no-caps
                      unelevated
                      color="warning"
                      text-color="dark"
                      v-for="(e,idx) in event_list"
                      :key ="idx"
                      :label="e.event_type"
                      
                    />
                  </div> -->
                  <q-toolbar class="" style="background:#FF9F7E; color:white;">
                    <q-toolbar-title>
                      <q-icon name="warning" />
                      Warning Bulletin Board
                    </q-toolbar-title>
                  </q-toolbar>

                  <div id="bulletin" ref="container">
                    <!-- <q-btn
                      no-caps
                      unelevated
                      color="warning"
                      text-color="dark"
                      @click="
                        add_btn('1', 'Cage-1 low volt', 'list', 'primary')
                      "
                      label="Insert Warning"
                    /> -->
                  </div>
                </q-card-section>
              </q-card>
            </div>

            <div class="col-3">
              <q-card style=" float:center;  text-align:center;">
                <q-card-section style="width:auto-fill; height:auto;">
                  <q-icon name="wifi" size="50px" >
                    <q-tooltip anchor="center right" self="center left" content-class="bg-indigo" :offset="[10, 10]" content-style="font-size:14px;">
                      Connection
                    </q-tooltip>
                  </q-icon>

                  <pie-chart
                    v-if="loaded"
                    :chartdata="doughnut_alive_device_data"
                    :options="alive_device_options"
                  />
                </q-card-section>
              </q-card>
            </div>

            <div class="col-3">
              <q-card style=" float:center;  text-align:center;">
                <q-card-section style="width:auto-fill; height:auto;">
                  <q-icon name="battery_charging_full" size="50px">
                    <q-tooltip anchor="center right" self="center left" content-class="bg-indigo" :offset="[10, 10]" content-style="font-size:14px;">
                      Battery
                    </q-tooltip>
                  </q-icon>
                  <pie-chart
                    v-if="loaded"
                    :chartdata="doughnut_battery_alert_data"
                    :options="battery_alert_options"
                  />
                </q-card-section>
              </q-card>
            </div>

            <div class="col-3">
              <q-card style=" float:center;  text-align:center;">
                <q-card-section style="width:auto-fill; height:auto;">
                  <q-icon name="gradient " size="50px" >
                    <q-tooltip anchor="center right" self="center left" content-class="bg-indigo" :offset="[10, 10]" content-style="font-size:14px;">
                      Environment
                    </q-tooltip>
                  </q-icon>
                  <pie-chart
                    v-if="loaded"
                    :chartdata="doughnut_dirt_alert_data"
                    :options="dirt_alert_options"
                  />
                </q-card-section>
              </q-card>
            </div>

            <div class="col-3">
              <q-card style=" float:center;  text-align:center;">
                <q-card-section style="width:auto-fill; height:auto;">
                  <q-icon name="local_dining  " size="50px">
                    <q-tooltip anchor="center right" self="center left" content-class="bg-indigo" :offset="[10, 10]" content-style="font-size:14px;">
                      Feed
                    </q-tooltip>
                  </q-icon>
                  <pie-chart
                    v-if="loaded"
                    :chartdata="doughnut_feeder_alert_data"
                    :options="feeder_alert_options"
                  />
                </q-card-section>
              </q-card>
            </div>

            <!-- <div class="col-6">
              <q-card style="width:350px; height:350px; float:right; right:2em; margin-top:25px; text-align:center;">
                <q-card-section style="width:350px; height:350px;">
                  <pie-chart v-if="loaded" :chartdata="chartdata" />
                </q-card-section>
              </q-card>
            </div>
            <div class="col-6 left">
              <q-card style="width:350px; height:350px; margin-top:25px; float:left; left:2em; text-align:center;">
                <q-card-section style="width:350px; height:350px;">
                  <pie-chart v-if="loaded" :chartdata="chartdata" />
                </q-card-section>
              </q-card>
            </div> -->
          </div>
        </q-carousel-slide>
        <template v-slot:control>
          <q-carousel-control position="bottom-right" :offset="[8, 8]">
            <q-btn
              push
              round
              dense
              size="18px"
              color="white"
              text-color="primary"
              :icon="fullscreen ? 'fullscreen_exit' : 'fullscreen'"
              @click="fullscreen = !fullscreen"
            >
              <q-tooltip anchor="center left" self="center right" content-class="bg-indigo" :offset="[10, 10]" content-style="font-size:10px;">
                Fullscreen
              </q-tooltip>
            </q-btn>
          </q-carousel-control>
        </template>

        
      </q-carousel>
    </div>
  </q-page>
</template>

<script>
import axios from "axios";
import LineChart from "../components/LineChart.js";
import PieChart from "../components/PieChart.js";
import moment from 'moment'
// import ChartJSPluginDatalabels from "chartjs-plugin-datalabels";
import ChartJspluginDoughnutlabel from "chartjs-plugin-doughnutlabel";
import Button from "../components/button";
import Vue from "vue";

var all_device_MAC = [];
var last_data_for_mqtt = [];

export default {
  name: "PageIndex",
  components: {
    LineChart,
    PieChart,
    Button
  },
  data() {
    return {
      okaudio: "",
      // all_device_MAC: [],
      slide: "overview",
      lorem: "Hi!",
      event_list: [
        {
          id: 1,
          rack_name: "mouse",
          cage_id: "3",
          event_type: "low volt"
        },
        {
          id: 2,
          rack_name: "mouse",
          cage_id: "4",
          event_type: "dead warning"
        }
      ],
      loaded: false,
      fullscreen: false,
      doughnut_alive_device_data: null,
      doughnut_battery_alert_data: null,
      doughnut_dirt_alert_data: null,
      doughnut_feeder_alert_data: null,
      chartdata: null,
      alive_device_options: {},
      battery_alert_options: {},
      dirt_alert_options: {},
      feeder_alert_options: {
        responsive: true,
        plugins: {
          doughnutlabel: {
            labels: []
          }
        }
      }
      //user_lastdata:[]
    };
  },
  computed: {

  },
  created() {
    // console.log("testing")
    console.log("all_device_MAC: ", all_device_MAC);
    this.okaudio = new Audio("https://mk100.frrut.net/when.mp3");
    this.loaded = false;
    this.get_api();
    
    this.topicmqtt = "topic/event/#";
    this.$mqtt.subscribe(this.topicmqtt);

    // this.intervalFetchData();
  },
  methods: {
    add_btn(
      rack_name = "",
      cage_id = "",
      text = "Click me!",
      icon_name = "",
      type = "",
      tooltip_text = "timeStr"
    ) {
      var ComponentClass = Vue.extend(Button);
      var instance = new ComponentClass({
        propsData: {
          rack_name: rack_name,
          _id: cage_id,
          type: type,
          icon_type: icon_name,
          tooltip_text: tooltip_text
        }
      });
      instance.$slots.default = [rack_name, " ", cage_id, " ", text];
      instance.$mount(); // pass nothing
      // console.log("this.$refs: ", this.$refs);
      this.$refs.container.appendChild(instance.$el);
    },
    del_btn(){
      const myNode = document.getElementById("bulletin");
      while (myNode.firstChild) {
        myNode.removeChild(myNode.lastChild);
      }
      // this.$refs.container.removeChild(instance.$el);
    },
    // triggerWarning(position) {
    //   this.$q.notify({
    //     type: "warning",
    //     message: 'Rack: Mouse, Cage: 56, "Feeder is Insufficient."',
    //     position,
    //     actions: [{ label: "OK", color: "red", handler: () => {} }]
    //   });
    // },
    playOKSound() {
      console.log("Play OK sound");
      const okaudioPromise = this.okaudio.play();
      if (okaudioPromise !== undefined) {
        okaudioPromise
          .then(() => {
            console.log("Finished Play OK sound");
            // Automatic playback started!
            // Show playing UI.
          })
          .catch(error => {
            // Auto-play was prevented
            // Show paused UI.
          });
      }
    },
    random_color(rand) {
      return "#" + Math.floor(rand * 16777215).toString(16);
    },
    checkTime() {
      return Date.parse(new Date());
    },
    // intervalFetchData() {
    //   setInterval(() => {
    //     this.get_api();
    //   }, 6000);
    // },
    get_api() {
      const requestOne = axios.get("https://api.frrut.net/device-collections");
      const requestTwo = axios.get("https://api.frrut.net/last-data");
      const requestThree = axios.get("https://api.frrut.net/mk-100-users");
      // const requestFour = axios.get("https://api.frrut.net/all-data?_limit=10000&_sort=timestamp:DESC");

      var nowtime = this.checkTime() / 1000;
      console.log("nowtime: ", nowtime, typeof nowtime);
      axios
        .all([requestOne, requestTwo, requestThree])
        .then(
          axios.spread((...responses) => {
            const deviceCollection = responses[0].data;
            const lastdata = responses[1].data.filter(function(item) {
              return item.MAC;
            });
            last_data_for_mqtt = lastdata
            const mk100users = responses[2].data;
            // const alldata = responses[3].data

            // console.log("deviceCollection: ", deviceCollection)
            // console.log("lastdata: ", lastdata)
            // console.log("mk100users: ", mk100users)
            // console.log("alldata: ", alldata)

            var userInfo = [JSON.parse(localStorage.getItem("loginInfo"))];

            for (var e = 0; e < userInfo.length; e++) {
              for (var f = 0; f < mk100users.length; f++) {
                if (userInfo[e].id === mk100users[f]._id) {
                  userInfo[e].role = mk100users[f].role;
                }
              }
            }

            var role = userInfo[0].role;
            all_device_MAC = [];
            var user_lastdata = [];
            console.log(role)

            if (role === "admin") {
              deviceCollection.map(function(a) {
                all_device_MAC.push(a.MAC);
              });

              lastdata.map(function(a) {
                if (all_device_MAC.includes(a.MAC)) {
                  user_lastdata.push(a);
                }
              });
            }
            else {
              deviceCollection.map(function(a) {
                if (a.registerUserID === userInfo[0].id) {
                  all_device_MAC.push(a.MAC);
                }
              });

              lastdata.map(function(a) {
                if (all_device_MAC.includes(a.MAC)) {
                  user_lastdata.push(a);
                }
              });
            }
            console.log("user_lastdatauser_lastdata: ", user_lastdata)

            for (var j = 0; j < user_lastdata.length; j++) {
              user_lastdata[j].timeStr = moment(user_lastdata[j].timestamp * 1000).locale('zh-TW').format('YYYY/MM/DD HH:mm:ss')
            }


            
            user_lastdata = user_lastdata.sort((a, b) => b.timestamp - a.timestamp)
            this.buffArray = user_lastdata

            var qtyMax = 200
            if (this.buffArray.length > qtyMax) {
              var temparray = []
              for (let i = 0; i < qtyMax; i++) {
                temparray.push(this.buffArray[i])
              }
              user_lastdata = temparray
              this.check_for_add_btn(user_lastdata)
            } 
            else {
              console.log(user_lastdata.length)
              this.check_for_add_btn(user_lastdata)
            } 


            var Timeout_devices =
              all_device_MAC.length - user_lastdata.length;

            var check_time_list = user_lastdata.map(
              item => nowtime - item.timestamp >= 14400
            );
            // console.log("check_time_list: ", check_time_list);

            for (var i = 0; i < check_time_list.length; i++) {
              if (check_time_list[i]) {
                // console.log("One device Timeout!");
                user_lastdata.pop();
                Timeout_devices += 1;
              }
            }
            console.log("user_lastdata: ", user_lastdata, user_lastdata.length);

            var Working_devices = user_lastdata.length;

            var Sufficient_battery = user_lastdata.filter(x => x.voltAlertColor === "green").length;
            var Insufficient_battery = user_lastdata.filter(x => x.voltAlertColor === "red").length;

            var Clean = user_lastdata.filter(x => x.blobAlertColor === "green").length;
            var Dirty = user_lastdata.filter(x => x.blobAlertColor === "red").length;

            var Sufficient_feed = user_lastdata.filter(x => x.feederAlertColor === "green").length;
            var Insufficient_feed = user_lastdata.filter(x => x.feederAlertColor === "red").length;

            this.alive_device_options = {
              responsive: true,
              plugins: {
                doughnutlabel: {
                  labels: [
                    {
                      text:
                        Timeout_devices +
                        " of " +
                        (Working_devices + Timeout_devices),
                      font: { size: "20", style: "bold" }
                    },
                    // {
                    //   text: "Timeout",
                    //   font: { size: "20", style: "bold" }
                    // }
                  ]
                }
              }
            };

            this.battery_alert_options = {
              responsive: true,
              plugins: {
                doughnutlabel: {
                  labels: [
                    {
                      text:
                        Insufficient_battery +
                        " of " +
                        (Sufficient_battery + Insufficient_battery),
                      font: { size: "20", style: "bold" }
                    },
                    // {
                    //   text: "Insufficient",
                    //   font: { size: "20", style: "bold" }
                    // }
                  ]
                }
              }
            };

            this.dirt_alert_options = {
              responsive: true,
              plugins: {
                doughnutlabel: {
                  labels: [
                    {
                      text: Dirty + " of " + (Clean + Dirty),
                      font: { size: "20", style: "bold" }
                    },
                    // {
                    //   text: "Dirty",
                    //   font: { size: "20", style: "bold" }
                    // }
                  ]
                }
              }
            };

            this.feeder_alert_options = {
              responsive: true,
              plugins: {
                doughnutlabel: {
                  labels: [
                    {
                      text:
                        Insufficient_feed +
                        " of " +
                        (Sufficient_feed + Insufficient_feed),
                      font: { size: "20", style: "bold" }
                    },
                    // {
                    //   text: "Insufficient",
                    //   font: { size: "20", style: "bold" }
                    // }
                  ]
                }
              }
            };

            this.doughnut_alive_device_data = {
              labels: ["Connected", "Lost"],
              datasets: [
                {
                  backgroundColor: ["#40bf40", "#e6e6e6"],
                  data: [Working_devices, Timeout_devices]
                }
              ]
            };

            this.doughnut_battery_alert_data = {
              labels: ["Full", "Empty"],
              datasets: [
                {
                  backgroundColor: ["#40bf40", "#ff5c33"],
                  data: [Sufficient_battery, Insufficient_battery]
                }
              ]
            };

            this.doughnut_dirt_alert_data = {
              labels: ["Clean", "Dirty"],
              datasets: [
                {
                  backgroundColor: ["#40bf40", "#ff5c33"],
                  data: [Clean, Dirty]
                }
              ]
            };

            this.doughnut_feeder_alert_data = {
              labels: ["Full", "Empty"],
              datasets: [
                {
                  backgroundColor: ["#40bf40", "#ff5c33"],
                  data: [Sufficient_feed, Insufficient_feed]
                }
              ]
            };
            this.loaded = true;
          })
        )
        .catch(e => {
          console.error(e);
        });
    },
    check_for_add_btn(check_data){

      console.log("check_data: ", check_data)
      this.del_btn()
      for(var i = 0; i < check_data.length; i++){
        if(check_data[i].feederAlert_str === "飼料不足"){
          this.add_btn(check_data[i].rackName, check_data[i].cageID, check_data[i].feederAlert_str, 'local_dining', 'warning', check_data[i].timeStr)
        }
        if (check_data[i].blobAlert_str === "髒污") {
          this.add_btn(check_data[i].rackName, check_data[i].cageID, check_data[i].blobAlert_str, 'gradient', 'warning', check_data[i].timeStr)
        }
        if (check_data[i].voltAlert_str === "電量不足") {
          this.add_btn(check_data[i].rackName, check_data[i].cageID, check_data[i].voltAlert_str, 'battery_alert', 'warning', check_data[i].timeStr)

        }
      }

    },
  },
  mounted() {},

  mqtt: {
    "topic/event/#"(mqtt_data_payload, topic) {
      // console.log("topic(this.topicmqtt): " + this.topicmqtt);
      this.obj = JSON.parse(mqtt_data_payload);
      console.log("this.obj: ", this.obj);

      this.timestr = this.obj.timeStr;
      console.log("all_device_MAC: ", all_device_MAC);
      // console.log('this.userMACList, userMAC')
      // console.log(this.userMACList)
      var checkMAC = all_device_MAC.filter(test => test === this.obj.MAC).toString()
      console.log("checkMAC: ", checkMAC, "   checkMAC.length: ", checkMAC.length)

      
      if (checkMAC.length === 12) {

        if(this.obj.feederAlert_str === "飼料不足" || this.obj.blobAlert_str === "髒污" || this.obj.voltAlert_str === "電量不足"){
          this.playOKSound()
        }

        this.buffArray = last_data_for_mqtt.filter(exclude => exclude.MAC !== checkMAC)
        console.log("this.buffArray before adding: ", this.buffArray)
        this.buffArray.unshift(this.obj)
        console.log("this.buffArray after adding: ", this.buffArray)

        for (var j = 0; j < this.buffArray.length; j++) {
          this.buffArray[j].timeStr = moment(this.buffArray[j].timestamp * 1000).locale('zh-TW').format('YYYY/MM/DD HH:mm:ss')
          this.buffArray[j].volt = parseFloat(this.buffArray[j].volt.toFixed(2))
        }

        var qtyMax = 200
        if (this.buffArray.length > qtyMax) {
          var temparray = []
          for (let i = 0; i < qtyMax; i++) {
            temparray.push(this.buffArray[i])
          }
          last_data_for_mqtt = temparray
          this.check_for_add_btn(last_data_for_mqtt)
          
        } 
        else {
          console.log(this.buffArray.length)
          last_data_for_mqtt = this.buffArray
          this.check_for_add_btn(last_data_for_mqtt)
        }
      } 
      else {
        console.log('not for this user, NO show')
      }
    }
  }
};
</script>

<style lang="scss">
.background-image {
  height: 100%;
  width: 50%;
  z-index: -1;
  background-position: left center;
  background-repeat: no-repeat;
  opacity: 0.6;
}
</style>
<style>
#app {
  font-family: "Avenir", Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: #2c3e50;
  margin-top: 60px;
}
</style>
