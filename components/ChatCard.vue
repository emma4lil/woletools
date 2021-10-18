<template>
  <v-card
    lazy-src="https://picsum.photos/id/11/10/6"
    flat
    class="ma-2 rounded-md"
    color="grey lighten-4"
  >
    <div
      class="caption ma-2"
    >
      <div class="d-flex justify-space-between">
        <div>{{data.ReceiverEmail}}</div>
        --
        <div>{{ data.SenderEmail }}</div>      
      </div>
    </div>
    <v-card-text v-if="!data.IsMedia" class="text-h6">
      <div class="d-flex" v-bind:class="{false: justify-end}">
        <div>{{ data.Message }}</div>
      </div>
    </v-card-text>
    <v-card-text v-if="data.IsMedia">
      <v-img
        lazy-src="https://picsum.photos/id/11/10/6"
        class="rounded-xl"
        :src="data.URI"
        contain
        width="200"
      >
      </v-img>
    </v-card-text>
    {{isSender}}
    <!-- <v-card-text>
      <v-chip small class=""> {{ data.CreatedAt }}</v-chip>
    </v-card-text> -->
  </v-card>
</template>

<script>
export default {
  name: "ChatCard",
  props: ["data", "sender"],
  computed: {
      isSender () {
          return this.data.SenderEmail === this.sender
      }
  }
};
</script>

<style>
</style>