<template>
  <div class="home-container">
    <div style="width:100%;height:100%;;">
      <p style="width:100%;text-align:center;color:rgb(225,225,225);">等待订单中</p>
    </div>
    <ReceiveOrder style="width:100%;height:100%;" />
  </div>
</template>

<script lang="ts">
// @ is an alias to /src
import LBSStat from "@/components/LBSStat/LBSStat.vue";
import ReceiveOrder from "@/components/ReceiveOrder/ReceiveOrder.vue";

import { Component, Vue, Prop } from "vue-property-decorator";
import { WSService } from "@/service/ws.service";
import { GeoLocation } from "@/components/LBSStat/LBSStat.vue";
import ApiService from "@/api/api.service";

@Component({
  components: {
    LBSStat,
    ReceiveOrder,
  },
})
export default class DriverHome extends Vue {
  private locs: GeoLocation[] = [];
  private sidPrefix = "lbsclient_";
  private asOpen: boolean = true;

  public mounted(): void {
    const user = uni.getStorageSync("user");
    const role = "" + uni.getStorageSync("userRole");
    WSService.initiate();
    WSService.connect(this.sidPrefix + new Date().getDate(), user.userID, role);
    WSService.msgSubject.subscribe((data: any) => {
      if (!(data instanceof Array)) {
        console.log("Unknown message:" + data);
        return;
      }
      this.locs = [];
      for (const ele of data) {
        const geo = ele.geoinfo.coordinates;
        console.log(geo);
        if (geo instanceof Array && geo.length == 2) {
          this.locs.push({ lng: geo[0], lat: geo[1] });
        }
      }
    });
    setInterval(() => {
      WSService.sendMessage(
        JSON.stringify({
          cmd: 3,
          role: 0,
          qrole: 1,
          geo: { lng: 116.404, lat: 39.915 },
          qgeo: { lng: 116.404, lat: 39.915 },
        })
      );
      WSService.sendMessage(JSON.stringify({ lng: 116.404, lat: 39.915 }));
    }, 5000);
  }

  onLoad() {
    // on callcar
  }

  onUnload() {
    uni.$off("callcar", () => {
      console.log("unsubscribe callcar event");
    });
  }
}
</script>

<style scoped>
.home-container {
  padding: 0px 0px 40px 0px;
  height: 100%;
  background-color: rgba(90, 131, 192, 0.8);
  position: relative;
}
</style>
