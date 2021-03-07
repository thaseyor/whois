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
            <b-input
              @keydown.enter.native="loadDomainInfo"
              v-model="domain"
              expanded
            ></b-input>
            <p class="control">
              <b-button
                type="is-primary"
                label="Search"
                @click="loadDomainInfo"
              /></p
          ></b-field>
          <div>
            <b-collapse
              v-for="item in domains"
              :key="item.title"
              class="card"
              animation="slide"
              aria-id="contentIdForA11y3"
              v-model="item.isActive"
            >
              <template #trigger="props">
                <div
                  class="card-header"
                  role="button"
                  aria-controls="contentIdForA11y3"
                >
                  <p class="card-header-title">
                    {{ item.title }}
                  </p>
                  <a class="card-header-icon">
                    <b-icon :icon="props.open ? 'menu-down' : 'menu-up'">
                    </b-icon>
                    <b-icon
                      icon="delete"
                      @click.stop.native="deleteDomain(item.title)"
                    >
                    </b-icon>
                  </a>
                </div>
              </template>

              <div class="card-content">
                <div class="content">
                  <pre>{{ item.message }}</pre>
                </div>
              </div>
            </b-collapse>
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
  created() {
    const domains = localStorage.getItem("domains-list");
    if (domains) {
      this.domains = JSON.parse(domains);
    }
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
      this.domains = [
        ...this.domains,
        {
          title: this.domain,
          message: jsonData["WhoisRecord"]["strippedText"]
            ? jsonData["WhoisRecord"]["strippedText"]
            : jsonData["WhoisRecord"]["registryData"]["strippedText"],
          isActive: false,
        },
      ];
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
    deleteDomain(domainToDeleteTitle) {
      this.domains = this.domains.filter(
        (d) => d.title !== domainToDeleteTitle
      );
    },
  },
  watch: {
    domains() {
      localStorage.setItem("domains-list", JSON.stringify(this.domains));
    },
  },
};
</script>

<style></style>
