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
        <b-tab-item label="IP"
          >
          <b-field>
            <b-input
              @keydown.enter.native="loadIpInfo"
              v-model="ip"
              expanded
            ></b-input>
            <p class="control">
              <b-button
                type="is-primary"
                label="Search"
                @click="loadIpInfo"
              /></p
          ></b-field>
          <p class="content">
            Your ip:
            <basefont @click="copyIp" style="cursor:pointer">
              {{ localip }}
            </basefont>
          </p>
          <div>
            <b-collapse
              v-for="item in ips"
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
                      @click.stop.native="deleteIp(item.title)"
                    >
                    </b-icon>
                  </a>
                </div>
              </template>

              <div class="card-content">
                <div class="content">
                  <p v-for="m in item.message" :key="m">{{ m }}</p>
                </div>
              </div>
            </b-collapse>
          </div></b-tab-item
        >
        <b-tab-item label="Phone number"
          ><b-field>
            <b-input
              @keydown.enter.native="loadPhoneInfo"
              v-model="phone"
              expanded
            ></b-input>
            <p class="control">
              <b-button
                type="is-primary"
                label="Search"
                @click="loadPhoneInfo"
              /></p
          ></b-field>
          <div>
            <b-collapse
              v-for="item in phones"
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
                      @click.stop.native="deletePhone(item.title)"
                    >
                    </b-icon>
                  </a>
                </div>
              </template>

              <div class="card-content">
                <div class="content">
                  <p v-for="m in item.message" :key="m">{{ m }}</p>
                </div>
              </div>
            </b-collapse>
          </div></b-tab-item
        >
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
      ip: "",
      phone: "",
      activeTab: 0,
      isLoading: false,
      domains: [],
      ips: [],
      localip: "",
      phones: [],
    };
  },
  async created() {
    const domains = localStorage.getItem("domains-list");
    if (domains) {
      this.domains = JSON.parse(domains);
    }

    const ips = localStorage.getItem("ips-list");
    if (ips) {
      this.ips = JSON.parse(ips);
    }

    const phones = localStorage.getItem("phones-list");
    if (phones) {
      this.phones = JSON.parse(phones);
    }

    const ipdata = await fetch(`https://ipinfo.io/json?token=2893d17b16a282`);
    const localip = await ipdata.json();
    this.localip = localip["ip"];
  },
  methods: {
    async loadDomainInfo() {
      if (this.domains.filter((d) => d.title === this.domain).length) {
        this.notify(`This domain has already been added!`, "is-danger");
        return;
      }
      if (!this.domain) {
        this.notify(`Empty string!`, "is-danger");
        return;
      }
      this.isLoading = true;
      const data = await fetch(
        `https://www.whoisxmlapi.com/whoisserver/WhoisService?apiKey=at_LcfLLteDNWP7Yq7vjlfUpWGMtrELY&outputFormat=JSON&domainName=${this.domain}`
      );
      const jsonData = await data.json();
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
    notify(message, type) {
      this.$buefy.toast.open({
        duration: 5000,
        message,
        type,
        queue: false,
      });
    },
    async loadIpInfo() {
      if (this.ips.filter((d) => d.title === this.ip).length) {
        this.notify(`This IP has already been added!`, "is-danger");
        return;
      }
      if (!this.ip) {
        this.notify(`Empty string!`, "is-danger");
        return;
      }
      this.isLoading = true;
      const data = await fetch(
        `https://ipinfo.io/${this.ip}/json?token=2893d17b16a282`
      );
      const jsonData = await data.json();
      this.isLoading = false;
      this.ips = [
        ...this.ips,
        {
          title: this.ip,
          message: [
            `Country: ${jsonData["country"]}`,
            `City: ${jsonData["city"]}`,
            `Region: ${jsonData["region"]}`,
            `Internet provider ${jsonData["org"]}`,
            `Hostname: ${jsonData["hostname"]}`,
            `Location: ${jsonData["loc"]}`,
            `Timezone: ${jsonData["timezone"]}`,
          ],
          isActive: false,
        },
      ];
      this.ip = "";
    },
    async loadPhoneInfo() {
      if (this.phones.filter((d) => d.title === this.phone).length) {
        this.notify(`This phone has already been added!`, "is-danger");
        return;
      }
      if (!this.phone) {
        this.notify(`Empty string!`, "is-danger");
        return;
      }
      this.isLoading = true;
      const data = await fetch(
        `https://htmlweb.ru/geo/api.php?telcod=${this.phone}&json`
      );
      const jsonData = await data.json();
      console.log(jsonData);
      this.isLoading = false;
      this.phones = [
        ...this.phones,
        {
          title: this.phone,
          message: [
            `Continent: ${jsonData["country"]['location']}`,
            `Country: ${jsonData["country"]['english']} (${jsonData['country']['id']})`,
            `Operator: ${jsonData[0]["oper"]} (${jsonData[0]["oper_brand"]})`,
          ],
          isActive: false,
        },
      ];
      this.phone = "";
    },
    deleteDomain(domainToDeleteTitle) {
      this.domains = this.domains.filter(
        (d) => d.title !== domainToDeleteTitle
      );
    },
    deleteIp(IpToDeleteTitle) {
      this.ips = this.ips.filter((i) => i.title !== IpToDeleteTitle);
    },
    deletePhone(PhoneToDeleteTitle) {
      this.phones = this.phones.filter((i) => i.title !== PhoneToDeleteTitle);
    },
    copyIp() {
      this.notify("IP copied to clipboard", "is-success");
      var copyText = this.localip;
      navigator.clipboard.writeText(copyText);
    },
  },
  watch: {
    domains() {
      localStorage.setItem("domains-list", JSON.stringify(this.domains));
    },
    ips() {
      localStorage.setItem("ips-list", JSON.stringify(this.ips));
    },
    phones() {
      localStorage.setItem("phones-list", JSON.stringify(this.phones));
    },
  },
};
</script>

<style></style>
