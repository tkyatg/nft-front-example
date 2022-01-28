<template>
  <v-container>
    <v-row>
      <v-col cols="12">
        <v-card>
          <v-list-item>
            <v-list-item-title> NFT作成 </v-list-item-title>
          </v-list-item>
          <v-list-item>
            <v-text-field v-model="uri" label="URI"></v-text-field>
          </v-list-item>
          <v-card-actions>
            <v-spacer></v-spacer>
            <v-btn rounded color="primary" large @click="createNft()">
              作成
            </v-btn>
          </v-card-actions>
        </v-card>
      </v-col>
    </v-row>
  </v-container>
</template>

<script>
import Web3 from "web3";
export default {
  data() {
    return {
      web3: null,
      nftContract: null,
      endpoint: "",
      contractAddress: "",
      uri: "",
    };
  },
  async created() {
    const web3 = new Web3(this.endpoint);
    const json = require("../abi/Nft.json");
    const nftContract = new web3.eth.Contract(json.abi, this.contractAddress);
    this.web3 = web3;
    this.nftContract = nftContract;
    try {
      const newAccounts = await ethereum.request({
        method: "eth_requestAccounts",
      });
      const accounts = newAccounts;
      if (accounts[0] !== "") {
        this.web3.eth.defaultAccount = accounts[0];
      } else {
        console.error("ログインに失敗");
      }
    } catch (error) {
      console.error(error);
    }
    await this.getMyNfts();
  },
  methods: {
    async getMyNfts() {
      const myNftCount = await this.nftContract.methods
        .balanceOf(this.web3.eth.defaultAccount)
        .call();
      for (let i = 0; i < myNftCount; i++) {
        let myNftTokenId = await this.nftContract.methods
          .tokenOfOwnerByIndex(this.web3.eth.defaultAccount, i)
          .call();
        let myNftTokenUri = await this.nftContract.methods
          .tokenURI(myNftTokenId)
          .call();
        console.log("あなたのNFT");
        console.log("tokenId: " + myNftTokenId);
        console.log("tokenUri: " + myNftTokenUri);
      }
    },
    async createNft() {
      const tx = {
        from: this.web3.eth.defaultAccount,
        to: this.contractAddress,
        data: this.nftContract.methods.mint(this.uri).encodeABI(),
      };
      try {
        await window.ethereum.request({
          method: "eth_sendTransaction",
          params: [tx],
        });
        alert("NFT作成成功");
      } catch (error) {
        alert("NFT作成失敗");
      }
    },
  },
};
</script>
