<template>
  <div id="app">
    <b-loading
      :is-full-page="true"
      v-model="isLoading"
      :can-cancel="false"
    ></b-loading>
    <section class="container">
      <b-tabs position="is-centered" class="block">
        <b-tab-item label="Domain"
          ><b-field>
            <b-input @keydown.enter="loadDomainInfo" v-model="domain"></b-input
            ><b-button label="Search" @click="loadDomainInfo"
          /></b-field>
          <div>
            <b-message v-for="item in domains" :key="item.name">
              <h2>{{ item.title }}</h2>
              {{ item.message }}
            </b-message>
          </div></b-tab-item
        >
        <b-tab-item label="IP"></b-tab-item>
        <b-tab-item label="Phone number"></b-tab-item>
      </b-tabs>
    </section>
  </div>
</template>

<script>
export default {
  name: "App",
  data() {
    return {
      domain: "",
      activeTab: 0,
      isLoading: false,
      domains: [],
    };
  },
  methods: {
    async loadDomainInfo() {
      if (this.domains.filter((d) => d.title === this.domain).length) {
        this.errorNotify(`This domain is already added!`);
        return;
      }
      if (!this.domain) {
        this.errorNotify(`Empty string!`);
        return;
      }
      this.isLoading = true;
      const data = await fetch(
        `https://www.whoisxmlapi.com/whoisserver/WhoisService?apiKey=at_LcfLLteDNWP7Yq7vjlfUpWGMtrELY&outputFormat=JSON&domainName=${this.domain}`
      );
      const jsonData = await data.json();
      console.log(jsonData);
      this.isLoading = false;
      this.domains.push({
        title: this.domain,
        message: `
        Nameserver 1: ${jsonData["WhoisRecord"]["registryData"]["nameServers"]["hostNames"][0]}
        \n
        Nameserver 2: ${jsonData["WhoisRecord"]["registryData"]["nameServers"]["hostNames"][1]}
      `,
      });
      this.domain = "";
    },
    errorNotify(message) {
      this.$buefy.toast.open({
        duration: 5000,
        message,
        type: "is-danger",
        queue: false,
      });
    },
  },
};
</script>

<style></style>
