<template>
  <div class="columns">
    <div class="column" style="padding: 0 20px">
      <b-field v-if="!fileToMint.name">
        <b-upload v-model="fileToMint" expanded drag-drop>
          <section class="section">
            <div class="content has-text-centered">
              <p>Drop your files here or click to upload</p>
            </div>
          </section>
        </b-upload>
      </b-field>
      <b-field label="Name">
        <b-input v-model="metadata.name"></b-input>
      </b-field>
      <b-field label="Description">
        <b-input
          maxlength="200"
          v-model="metadata.description"
          type="textarea"
        ></b-input>
      </b-field>
      <b-field label="Trait type">
        <b-input v-model="metadata.attributes[0].trait_type"></b-input>
      </b-field>
      <b-field label="Trait value">
        <b-input v-model="metadata.attributes[0].value"></b-input>
      </b-field>
      <div v-if="fileToMint.name">
        Selected file: <b>{{ fileToMint.name }}</b>
      </div>
      <div v-if="isMinting">Minting NFT, please wait...</div>
      <div v-if="isUploadingIPFS">Uploading file to IPFS, please wait...</div>
      <div v-if="isUploadingMetadata">
        Uploading metadata to IPFS, please wait...
      </div>
    </div>
    <div class="column">
      <pre style="text-align: left">{{
        JSON.stringify(metadata, null, 4)
      }}</pre>
      <b-button @click="uploadMetadata" style="width:100%; margin: 15px 0">UPLOAD METADATA TO IPFS</b-button>
      <div v-if="ipfsMetadata">
        Final metadata for your file is:<br />
        <a :href="'https://ipfs.io/ipfs/' + ipfsMetadata" target="_blank">{{ ipfsMetadata }}</a>
      </div>
    </div>
  </div>
</template>

<script>
const axios = require("axios");
const FormData = require("form-data");

export default {
  name: "Mint",
  data() {
    return {
      fileToMint: {},
      isUploadingIPFS: false,
      isUploadingMetadata: false,
      isMinting: false,
      axios: axios,
      ipfsMetadata: "",
      metadata: {
        name: "",
        description: "",
        image: "",
        attributes: [
          {
            trait_type: "",
            value: "",
          },
        ],
      },
      infuraURL: "https://ipfs.infura.io:5001/api/v0/add",
    };
  },
  watch: {
    fileToMint() {
      this.uploadFile();
    },
  },
  methods: {
    async uploadFile() {
      const app = this;
      if (app.fileToMint.name && !app.isUploadingIPFS) {
        app.isUploadingIPFS = true;
        const formData = new FormData();
        formData.append("file", app.fileToMint);
        axios({
          method: "post",
          url: app.infuraURL,
          data: formData,
          headers: {
            "Content-Type": "multipart/form-data",
          },
        }).then(function (response) {
          app.metadata.image = response.data.Hash;
          app.isUploadingIPFS = false;
        });
      } else {
        alert("Select a file first!");
      }
    },
    async uploadMetadata() {
      const app = this;
      app.isUploadingMetadata = true;
      const formData = new FormData();
      formData.append("metadata", JSON.stringify(app.metadata));
      axios({
        method: "post",
        url: app.infuraURL,
        data: formData,
        headers: {
          "Content-Type": "multipart/form-data",
        },
      }).then(function (response) {
        app.ipfsMetadata = response.data.Hash;
        app.isUploadingMetadata = false;
      });
    },
  },
};
</script>