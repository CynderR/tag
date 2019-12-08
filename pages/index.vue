<template>
  <div>
    <b-container>
      <b-row>
        <b-col class="mt-2" cols="12" v-show="videoLoaded === false">
          <b-form-file
            v-model="file"
            :state="Boolean(file)"
            placeholder="Choose a file or drop it here..."
            drop-placeholder="Drop file here..."
            v-on:change="parseFileInfo"
            class="droparea"
          ></b-form-file>
        </b-col>
      </b-row>
      <b-row v-show="videoLoaded" class="pt-2">
        <v-video-player :file="file"></v-video-player>
      </b-row>
      <b-row v-show="videoLoaded">
        <b-col class="text-center" cols="2">Tape Number</b-col>
        <b-col class="text-center" cols="2" placeholder="yyyy-mm-dd">Date</b-col>
        <b-col class="text-center" cols="2">Location</b-col>
        <b-col class="pl-4" cols="6">Tape Content</b-col>
      </b-row>

      <b-row v-show="videoLoaded">
        <b-col cols="2"><b-form-input v-model="fileNumber" placeholder="File number"></b-form-input></b-col>
        <b-col cols="2"><b-form-input v-on:change="fetchData" v-model="date" placeholder="Date"></b-form-input></b-col>
        <b-col cols="2"><b-form-input v-on:change="fetchData" v-model="location" placeholder="Location"></b-form-input></b-col>
        <b-col cols="6"><b-form-input v-model="tapeContent" placeholder="Details"></b-form-input></b-col>
      </b-row>

      <b-row v-show="videoLoaded" class="alert-danger bold m-2 p-2 font-weight-bold" style="font-size: x-large;">
        It's very important to keep the date in the format yyyy-mm-dd and to add the location.
      </b-row>

      <v-tag-modal
        v-show="videoLoaded"
        v-on:add-tag="addRowToTable"
        :setTimeStamp="setTimeStamp"
        :getVideosTimestamp="getVideosTimestamp"
      > </v-tag-modal>

<!-- Tag table -->
      <b-table
        v-show="videoLoaded"
        striped
        hover
        ref="tagTable"
        :fields="fields"
        :items="items"
        :key="items.timestamp"
      >
        <template v-slot:cell(formattedStart)="data">
          {{data.item.formattedStart}}
        </template>

        <template v-slot:cell(formattedEnd)="data">
          {{data.item.formattedEnd}}
        </template>

        <template v-slot:cell(tagLength)="data">
          {{data.item.tagLength}}
        </template>

        <template v-slot:cell(comment)="data">
          {{data.item.comment}}
        </template>

        <template v-slot:cell(tags)="data">
          {{data.value}}
        </template>

        <template v-slot:cell(setEnd)="data">
          <b-button @click="setEndTime(data.item)">End</b-button>
        </template>
      </b-table>
    </b-container>
  </div>
</template>

<script>
import TagModal from '~/components/TagModal.vue';
import VideoPlayer from '~/components/VideoPlayer.vue';
import axios from '~/plugins/axios';

export default {
  components: {
    'v-tag-modal': TagModal,
    'v-video-player': VideoPlayer
  },
  data() {
    return {
      file: null,
      fileNumber: null,
      location: null,
      date: null,
      tapeContent: null,
      videoLoaded: false,
      items: [],
      fields: [
        {
          key: 'formattedStart',
          label: 'Time Start',
          'sort-desc': 'false'
        },
        {
          key: 'formattedEnd',
          label: 'Time End'
        },
        {
          key: 'tagLength',
          label: 'Length'
        },
        {
          key: 'comment',
          label: 'comments'
        },
        {
          key: 'tags',
          label: 'Tags',
          class: 'table-tags',
          formatter: 'tagNames'
        },
        {
          key: 'setEnd',
          label: 'Set End',
          class: 'table-button'
        }
      ]
    };
  },
  methods: {
    timestamp(tags) {
      return tags[0].timestamp;
    },
    setEndTime(item) {
      item.tagEnd = this.getVideosTimestamp();
      item.formattedEnd = this.setTimeStamp(item.tagEnd);
      item.tagLength = this.setTimeStamp(item.tagEnd - item.tagStart);
      this.$refs.tagTable.refresh();
      this.savePage();
    },
    setTimeStamp(currentTime) {
      const seconds = String(Math.floor(currentTime % 60)).padStart(2, 0);
      const minuites = String(Math.floor(currentTime / 60 % 60)).padStart(2, 0);
      const hours = Math.floor(currentTime / 60 / 60);
      return `${hours ? hours+':' : ''}${minuites}:${seconds}`;
    },
    getVideosTimestamp() {
      if (document.getElementById('videoPlayer')) {
        return document.getElementById('videoPlayer').currentTime;
      }
      return false;
    },
    tagNames(tags) {
      return tags.map((item) => {
        return item.text;
      }).join(', ');
    },
    addRowToTable(row) {
      this.items.push(row);
      this.savePage();
    },
    parseFileInfo(event) {
      if ((!event.target.files || !event.target.files[0]) &&
      (!event.dataTransfer.files || !event.dataTransfer.files[0])) {
        return '';
      }
      const files = (event.target.files && event.target.files.length) ? event.target.files : event.dataTransfer.files;
      const splitName = files[0].name.split(' - ');
      this.fileNumber = splitName.shift();
      this.date = splitName.shift();
      this.tapeContent = splitName
        .join(' - ')
        .split('.')
        .slice(0, -1)
        .join('.');
      this.videoLoaded = true;
      this.fetchData();
    },
    fetchData() {
      axios({
        method: 'get',
        url: '/tags/' + this.date + this.location
      }).then((res) => {
        this.items = [];
        console.log('asd');
        for (let tag in res.data.tags) {
          res.data.tags[tag].timestamp = res.data.tags[tag].tagStart;
          this.items.push(res.data.tags[tag]);
          console.log('test');
        }
      }).catch((err) => {
        console.log(err);
        this.items = [];
      });
    },
    savePage() {
      let data = {
        id: this.date + this.location,
        tapeNumber: this.fileNumber,
        date: this.date,
        description: this.tapeContent,
        tags: this.items
      };
      axios({
        method: 'post',
        url: '/tags',
        data
      });

      console.log('save', data);
    }
  }
}
</script>
