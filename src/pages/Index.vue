<template>
  <q-page class="justify-center row window-height">
    <div>
       <p class="q-mt-md">Disclaimer: This tool is for private use only to review,
        critise and report NewsCorp's front pages.</p>
      <div class="q-gutter-sm">
        <q-checkbox v-for="brand of brands" :key="brand.name"
          v-model="publications"
          :val="brand.value"
          :label="brand.name"
        />
      </div>
      <q-input v-model="startDate" filled type="date" hint="Starting Date" mask="date">
        <template v-slot:after>
          <q-btn round dense flat icon="send" @click="loadDate"/>
        </template>
      </q-input>
      <q-infinite-scroll @load="onLoad" :offset="250" v-if="publications.length != 0">
        <div v-for="date in dates" :key="`${date}`" class="q-mb-md">
          <div class="full-width row">
          <q-img v-for="brand of publications" :key="brand"
            :src="`https://mfeeds.news.com.au/smedia/${brand}_1_${dateFormat.formatDate(date, 'YYYY_MM_DD')}_thumb_big.jpg`"
            class="col-4"
            @click="window.open(`https://mfeeds.news.com.au/smedia/${brand}_1_${dateFormat.formatDate(date, 'YYYY_MM_DD')}_thumb_big.jpg`, '_blank');"
          />
          </div>
          <h6 class="q-ma-none text-center">
            {{dateFormat.formatDate(date, 'dddd MMMM Do YYYY')}}
          </h6>
        </div>
        <template v-slot:loading>
          <div class="row justify-center q-my-md">
            <q-spinner-dots color="primary" size="40px" />
          </div>
      </template>
      </q-infinite-scroll>
    </div>
  </q-page>
</template>

<script>
import { date } from 'quasar';

const brands = [
  {
    name: 'The Australian',
    value: 'AUSTRALIAN/NCAUS',
  },
  {
    name: 'Daily Telegraph',
    value: 'TELEGRAPH/NCTELE',
  },
  {
    name: 'Herald Sun',
    value: 'HERALDSUN/NCHRS',
  },
  {
    name: 'Courier Mail',
    value: 'NCCOURIER/NCCM',
  },
  {
    name: 'The Advertiser',
    value: 'ADVERTISER/NCADV',
  },
  {
    name: 'Mercury',
    value: 'MERCURY/NCMER',
  },
  {
    name: 'Gold Coast Bulletin',
    value: 'GOLDCOASTBULLETIN/NCGCB',
  },
];
export default {
  name: 'PageIndex',
  data() {
    return {
      publications: [],
      brands,
      startDate: date.formatDate(new Date(), 'YYYY-MM-DD'),
      dates: [new Date()],
      dateFormat: date,
      window,
    };
  },
  mounted() {
    const queryString = window.location.search;
    const urlParams = new URLSearchParams(queryString);
    const params = urlParams.get('publications');
    if (params && params.split(',')) {
      this.publications = params.split(',');
    }
  },
  methods: {
    loadDate() {
      const sDate = date.extractDate(this.startDate, 'YYYY-MM-DD');
      if (!this.startDate || !sDate) {
        this.$q.notify('Please enter a valid date.');
        return;
      }
      this.dates = [sDate];
      this.onLoad(null, () => {});
      if (this.publications.length === 0) {
        this.$q.notify('Please select a publication.');
      }
    },
    onLoad(index, done) {
      const lastDate = this.dates[this.dates.length - 1];
      let addedCount = 1;
      while (addedCount < 5) {
        this.dates.push(new Date(lastDate.getTime() - addedCount * 60 * 60 * 24 * 1000));
        addedCount += 1;
      }
      done();
    },
  },
  watch: {
    publications(newVal) {
      this.$router.push({ query: { publications: newVal.join(',') } });
    },
  },
};
</script>
