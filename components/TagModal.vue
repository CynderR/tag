<template>
  <div>
    <b-row class="pt-5">
      <b-col><b-button @click="openModal()">Add Tag's</b-button></b-col>
    </b-row>
    <b-modal id="modal-tag" title="BootstrapVue" @ok="addTag">
      <p class="my-4">
        <span>Tag's start time: {{ formattedStart }}</span>
        <vue-tags-input
          v-model="tag"
          :tags="tags"
          @tags-changed="newTags => tags = newTags"
        />
        <br>
        <b-form-input v-model="comment" placeholder="comments"></b-form-input>
      </p>
    </b-modal>
  </div>
</template>
<script>
import VueTagsInput from '@johmun/vue-tags-input';
// import axios from 'axios';

export default {
  components: {
    VueTagsInput
  },
  props: {
    setTimeStamp: {
      type: Function
    },
    getVideosTimestamp: {
      type: Function
    }
  },
  data () {
    return {
      currentTime: null,
      tag: '',
      tags: [],
      comment: '',
      autocompleteItems: [],
      formattedStart: null
    }
  },
  // asyncData(context) {
  //   return axios.get('/tags')
  //     .then((res) => {
  //       if (res.data[0]) {
  //         console.log('data', res.data);
  //       }
  //     })
  //     .catch((err) => {
  //       console.log('error', err);
  //     });
  // },
  methods: {
    addTag() {
      const tableRow = {
        tagStart: this.currentTime,
        formattedStart: this.formattedStart,
        tagEnd: '',
        formattedEnd: '',
        tags: this.tags,
        comment: this.comment
      };
      this.$emit('add-tag', tableRow);
    },

    openModal() {
      this.tags = [];
      this.comment = '';
      this.currentTime = this.getVideosTimestamp();
      if (this.currentTime === false) {
        return;
      }
      this.formattedStart = this.setTimeStamp(this.currentTime);
      this.$root.$emit('bv::show::modal', 'modal-tag');
    }
  }
};
</script>
