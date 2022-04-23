<template>
  <div>
    <div v-if="!ipfsMetadata" class="columns">
      <div class="column" style="padding: 0 20px">
        <img
          v-if="metadata.image"
          :src="metadata.image.replace('ipfs://', 'https://ipfs.io/ipfs/')"
        /><br />
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
        <div style="padding: 0 10px">
          <pre style="text-align: left">{{
            JSON.stringify(metadata, null, 4)
          }}</pre>
          <b-button @click="uploadMetadata" style="width: 100%; margin: 15px 0"
            >UPLOAD METADATA TO IPFS</b-button
          >
          <div v-if="ipfsMetadata">
            Final metadata for your file is:<br />
            <a :href="'https://ipfs.io/ipfs/' + ipfsMetadata" target="_blank">{{
              ipfsMetadata
            }}</a>
          </div>
        </div>
      </div>
    </div>
    <div v-if="ipfsMetadata" style="text-align: center">
      <h1 class="title is-1">Publish NFT on Contract</h1>
      If you have a contract address please write it down here or simply create
      a new contract now.
      <div v-if="account">
        Now using {{ account }} address.<br />
        <b-field label="Contract address">
          <b-input v-model="contractAddress"></b-input>
        </b-field>
        <b-button
          type="is-primary"
          v-if="!contractAddress && !isWorking"
          v-on:click="createContract"
          >CREATE CONTRACT</b-button
        >
        <b-button
          type="is-primary"
          v-if="contractAddress && !isWorking"
          v-on:click="mint"
          >MINT NFT</b-button
        >
        <div v-if="isWorking">{{ workingMessage }}</div>
      </div>
      <div v-if="!account">
        Please connect your Metamask wallet first,<br />window should be open
        automatically or click below button.<br /><br />
        <b-button type="is-primary" v-on:click="connect"
          >CONNECT METAMASK</b-button
        >
      </div>
    </div>
  </div>
</template>

<script>
const axios = require("axios");
const FormData = require("form-data");
const ArtifactContract = require("@/SimpleNFT.json");
const Contract = require("@truffle/contract");
var Web3 = require("web3");

export default {
  name: "Mint",
  data() {
    return {
      web3: new Web3(window.ethereum),
      fileToMint: {},
      isUploadingIPFS: false,
      isUploadingMetadata: false,
      isMinting: false,
      axios: axios,
      ipfsMetadata: "",
      contractAddress: "",
      account: "",
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
      isWorking: false,
      workingMessage: "",
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
          app.metadata.image = "ipfs://" + response.data.Hash;
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
    async createContract() {
      const app = this;
      if (app.account.length > 0 && !app.isWorking) {
        app.isWorking = true;
        app.workingMessage =
          "Creating contract, please confirm with Metamask..";
        const Artifact = Contract(ArtifactContract);
        Artifact.setProvider(window.ethereum);
        try {
          const instance = await Artifact.new("SimpleNFT", "sNFT", {
            from: app.account,
            gas: 8000000,
          });
          console.log("Created new contract:", instance.address);
          app.contractAddress = instance.address;
          alert("Contract created successfully!");
          app.isWorking = false;
        } catch (e) {
          console.log(e);
          app.isWorking = false;
          alert(e.message);
        }
      }
    },
    async connect() {
      const app = this;
      window.ethereum.enable();
      try {
        let accounts = await this.web3.eth.getAccounts();
        app.account = accounts[0];
      } catch (e) {
        alert(e.message);
      }
    },
    async mint() {
      const app = this;
      console.log("Minting NFT..");
      if (!app.isWorking) {
        app.isWorking = true;
        app.workingMessage = "Minting NFT, please confirm with Metamask..";
        const contract = await new this.web3.eth.Contract(
          ArtifactContract.abi,
          this.contractAddress,
          {
            gasLimit: "5000000",
          }
        );
        try {
          await contract.methods
            .mint(app.ipfsMetadata)
            .send({ from: app.account });
          app.isWorking = false;
          alert("NFT minted correctly!");
          app.metadata = {
            name: "",
            description: "",
            image: "",
            attributes: [
              {
                trait_type: "",
                value: "",
              },
            ],
          };
        } catch (e) {
          alert(e.message);
          app.isWorking = false;
        }
      } else {
        console.log("App is busy..");
      }
    },
  },
};
</script>