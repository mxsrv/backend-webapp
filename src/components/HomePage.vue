<template>
  <v-container fluid grid-list-md>
    <v-row no-gutters>
      <v-col>
        <v-card
          class="mx-auto pa-4"
          outlined
          title
          width="900px"
          height="900px"
          elevation="10"
        >
          <div class="first-row">
            <span
              style="
                font-size: 25px;
                font-family: Arial;
                color: rgb(0, 0, 0);
                background-color: transparent;
                padding: 10px;
              "
            >
              Temperature Plot
            </span>
          </div>
          <v-divider class="ma-2"></v-divider>
          <v-card elevation="10" height="779px">
            <v-img
              alt="No temperature data for today"
              class="shrink mr-2"
              contain
              src="http://10.3.142.1:8000/api/getTemperaturePlot/2"
              max-width="850"
              max-height="850"
            />
          </v-card>
        </v-card>
      </v-col>
      <v-col>
        <v-card
          class="mx-auto pa-4"
          outlined
          title
          width="900px"
          height="900px"
          elevation="10"
        >
          <div class="first-row">
            <span
              style="
                font-size: 25px;
                font-family: Arial;
                color: rgb(0, 0, 0);
                background-color: transparent;
                padding: 10px;
              "
            >
              Data on Backend Server
            </span>
            <v-spacer></v-spacer>
            <v-select
              v-model="piSelected"
              :items="rasperryPiNum"
              label="Choose a Sensorbox"
              outlined
              v-on:change="getFiles"
            ></v-select>
          </div>
          <v-divider></v-divider>
          <div class="first-row">
            <span
              style="
                font-size: 22px;
                font-family: Arial;
                color: rgb(0, 0, 0);
                background-color: transparent;
                padding: 5px;
              "
            >
              Files
            </span>
          </div>
          <v-divider class="mb-2"></v-divider>
          <v-card elevation="10" height="701px">
            <v-simple-table height="701px">
              <template v-slot:default>
                <thead>
                  <tr>
                    <th class="text-left">Filename</th>
                    <th class="text-left">More Info</th>
                    <th class="text-left">Image</th>
                    <th class="text-left">Received Chunks</th>
                  </tr>
                </thead>
                <tbody>
                  <tr v-for="item in files" :key="item.index">
                    <td>{{ item.filename }}</td>
                    <td>
                      <v-btn
                        color="green lighten-1"
                        text
                        @click="$set(infoDialog, item.index, true)"
                      >
                        Info</v-btn
                      >
                      <v-dialog
                        v-model="infoDialog[item.index]"
                        scrollable
                        max-width="500"
                        :key="item.index"
                      >
                        <v-card>
                          <v-card-title>
                            <span> Details about {{ item.filename }}</span>
                          </v-card-title>
                          <v-card-text>
                            <v-container>
                              <v-row
                                >Content Type: {{ item.contentType }}</v-row
                              >
                              <v-row>Upload Date: {{ item.uploadDate }}</v-row>
                              <v-row
                                >Number of Chunks: {{ item.chunksNum }}</v-row
                              >
                            </v-container>
                          </v-card-text>
                          <v-card-actions>
                            <v-btn
                              color="green lighten-1"
                              text
                              @click.stop="$set(infoDialog, item.index, false)"
                            >
                              Close
                            </v-btn>
                          </v-card-actions>
                        </v-card>
                      </v-dialog>
                    </td>
                    <td>
                      <v-img
                        :src="
                          'http://10.3.142.1:8000/api/getGridfsImage/' +
                          item.id
                        "
                        alt="No Image"
                        height="234px"
                        width="354px"
                      ></v-img>
                    </td>
                    <td>{{ chunkCounts[item.id] }}/{{ item.chunksNum }}</td>
                  </tr>
                </tbody>
              </template>
            </v-simple-table>
          </v-card>
        </v-card>
      </v-col>
    </v-row>
  </v-container>
</template>

<script>
import axios from "axios";

export default {
  data() {
    return {
      files: [],
      rasperryPiNum: [],
      chunkCounts: [],
      infoDialog: {},
      piSelected: 2,
      prevPi: 2,
      boxRawIP: "10.3.142.1:8000",
      boxRawIP2: "10.3.142.174:8000",
    };
  },

  created() {
    this.getNumberOfChunks();
    this.getPis();
    this.getFiles();
  },

  mounted: function () {
    this.$nextTick(function () {
      window.setInterval(() => {
        this.getNumberOfChunks();
        this.getFiles();
      }, 1000);
    });
  },

  methods: {
    getPis() {
      this.rasperryPiNum = [0, 1, 2, 3, 4];
    },
    getFiles() {
      if (this.prevPi != this.piSelected) {
        this.files = [];
        this.prevPi = this.piSelected;
      }
      this.$http
        .get(
          "http://" +
            this.boxRawIP +
            "/api/listFiles/" +
            this.piSelected.toString()
        )
        .then(async (res) => {
          if (res.status == 200) {
            var i;
            for (i = 0; i < res.body.length; i = i + 1) {
              if (!this.files.some((item) => item.id == res.body[i]._id)) {
                this.files.push({
                  index: i,
                  id: res.body[i]._id,
                  filename: res.body[i].filename,
                  contentType: res.body[i].contentType,
                  uploadDate: new Date(res.body[i].uploadDate).toLocaleString(),
                  chunksNum: Math.ceil(
                    res.body[i].length / res.body[i].chunkSize
                  ),
                });
              } else {
                this.files[i].chunksNum = Math.ceil(
                  res.body[i].length / res.body[i].chunkSize
                );
              }

              if (
                this.files[i].chunksNum == this.chunkCounts[this.files[i].id]
              ) {
                this.$http.get(
                  "http://" +
                    this.boxRawIP2 +
                    "/api/arrivedAtBackend/" +
                    this.files[i].id.toString()
                );
              }
            }
          }
        });
    },
    getNumberOfChunks() {
      this.$http
        .get(
          "http://" +
            this.boxRawIP +
            "/api/listChunks/" +
            this.piSelected.toString()
        )
        .then(async (res) => {
          var i;
          var fileIds = [];
          let counts = [];
          for (i = 0; i < res.body.length; i = i + 1) {
            fileIds.push(res.body[i].files_id);
          }
          for (i = 0; i < fileIds.length; i++) {
            if (counts[fileIds[i]]) {
              counts[fileIds[i]] += 1;
            } else {
              counts[fileIds[i]] = 1;
            }
          }
          this.chunkCounts = counts;
        });
    },
    downloadFile(id, filename) {
      axios({
        url:
          "http://" +
          this.boxRawIP +
          "/api/downloadFile/" +
          this.piSelected.toString() +
          "/" +
          id,
        method: "GET",
        responseType: "blob",
      }).then((res) => {
        var fileURL = window.URL.createObjectURL(new Blob([res.data]));
        var fileLink = document.createElement("a");

        fileLink.href = fileURL;
        fileLink.setAttribute("download", filename);
        document.body.appendChild(fileLink);

        fileLink.click();
      });
    },
  },
};
</script>

<style lang="scss" scoped>
.first-row {
  margin: 5px 0;
  display: flex;
  align-items: center;
}
</style>
