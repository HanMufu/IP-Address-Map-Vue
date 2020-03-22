<template>
  <div class="table">
    <div class="container">
      <el-row :justify="center" gutter=20>

        <el-col :span="4">
          <el-input
            type="text"
            placeholder="IP address"
            v-model="ip_address"
            maxlength="15"
            show-word-limit
          >
          </el-input>
        </el-col>

        <el-col :span="7.5">
          <!-- <span class="demonstration">默认</span> -->
          <el-date-picker
            v-model="value1"
            value-format="timestamp"
            type="datetimerange"
            range-separator="to"
            start-placeholder="start from"
            end-placeholder="end to">
          </el-date-picker>
        </el-col>

        <el-col :span="4">
          <el-input v-model="last_n" placeholder="last N locations"></el-input>
        </el-col>

        <el-col :span="1">
          <el-button icon="el-icon-search" circle @click="getLocation"></el-button>
        </el-col>
      </el-row>

      <div style="margin: 5px 0;"></div>


      <el-row :justify="center">

        <gmap-map
          :center="center"
          :zoom="8"
          style="width:100%;  height: 600px;"
        >
          <gmap-marker
            :key="index"
            v-for="(m, index) in markers"
            :position="m.position"
            @click="center=m.position"
          ></gmap-marker>
        </gmap-map>

      </el-row>

      <div style="margin: 5px 0;"></div>

      <el-row :justify="center">
        <el-table
          :data="tableData"
          height="350"
          border
          style="width: 100%">
          <el-table-column
            prop="IPAddress"
            label="IP address"
            width="180">
          </el-table-column>
          <el-table-column
            prop="date"
            label="date"
            width="250">
          </el-table-column>
          <el-table-column
            prop="lon"
            label="longitude"
            width="180">
          </el-table-column>
          <el-table-column
            prop="lat"
            label="latitude">
          </el-table-column>
        </el-table>
      </el-row>

      <div style="margin: 5px 0;"></div>
        
      <el-row :justify="center">
        <el-col :span="10">
          <el-input placeholder="Phone Number" v-model="phoneNumber" class="input-with-select">
            <el-select v-model="areaCode" slot="prepend" placeholder="area code">
              <el-option label="+1" value="+1"></el-option>
              <el-option label="+86" value="+86"></el-option>
            </el-select>
            <el-button slot="append" icon="el-icon-s-promotion" @click="sendSMS"></el-button>
          </el-input>
        </el-col>
      </el-row>

    </div>
  </div>
</template>


<script>
export default {
  name: "GoogleMap",
  data() {
    return {
      // default to Montreal to keep it simple
      // change this to whatever makes sense
      urlGenerate: "/location/history", 
      urlSendSMS: "/SMS/send", 
      center: { lat: 45.508, lng: -73.587 },
      markers: [],
      places: [],
      currentPlace: null, 
      // value1: [new Date(2020, 2, 20, 12, 30), new Date(2020, 2, 21, 12, 30)],
      value1: [], 
      last_n: '', 
      ip_address: '18.191.20.188', 
      tableData: [], 
      phoneNumber: '', 
      areaCode: ''
    };
  },

  mounted() {
    this.geolocate();
  },

  methods: {
    sendSMS() {
      this.$axios.post(this.urlSendSMS, {
        phoneNumber: this.phoneNumber, 
        areaCode: this.areaCode, 
        message: this.tableData
      }).then(res => {
        this.$message(res.data.alert);
      });
    }, 
    getLocation() {
      this.markers = [];
      this.$axios.get(this.urlGenerate, {
        params: {
          ip_address: this.ip_address, 
          start_date: this.value1[0], 
          end_date: this.value1[1], 
          last_n: this.last_n
        }
      }).then(res => {
        this.tableData = res.data.records;
        this.markers = [];
        for(var i = 0; i < res.data.records.length; i++) {
          console.log(typeof res.data.records[i].lat);
          const marker = {
            lat: Number(res.data.records[i].lat), 
            lng: Number(res.data.records[i].lon)
          };
          this.markers.push({position: marker});
          this.center = marker;
        };
      });
    }, 
    // receives a place object via the autocomplete component
    setPlace(place) {
      this.currentPlace = place;
    },
    addMarker() {
      if (this.currentPlace) {
        const marker = {
          lat: this.currentPlace.geometry.location.lat(),
          lng: this.currentPlace.geometry.location.lng()
        };
        this.markers.push({ position: marker });
        this.places.push(this.currentPlace);
        this.center = marker;
        this.currentPlace = null;
      }
    },
    // clearMarkers(){
    //   for (var i = 0; i < this.markers.length; i++){
    //     this.markers[i].setMap(null);
    //   }
    //   this.markers = [];
    // },
    geolocate: function() {
      navigator.geolocation.getCurrentPosition(position => {
        this.center = {
          lat: position.coords.latitude,
          lng: position.coords.longitude
        };
      });
    }
  }
};
</script>

<style>
  .el-select .el-input {
    width: 130px;
  }
  .input-with-select .el-input-group__prepend {
    background-color: #fff;
  }
</style>