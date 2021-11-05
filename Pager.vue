<template>
  <div class="pagerContainer">
    <div class="pagerDataContainer">
      <div class="messageInABottle">
        {{ message }}
      </div>
      <br />
      <textarea maxlength="255" v-model="pagerMessage"></textarea>
      <button class="pagerButton button" v-on:click="wave">📟 Beep</button>
      <ul class="info">
        <li>
          <span
            >Total Messages: {{ waveCount }} | Powered by Rinkeby Test
            Network</span
          >
        </li>
        <li>
          <div class="walletConnect">
            <button class="primaryButton button" v-on:click="connectWallet">
              Connect Wallet
            </button>
          </div>
        </li>
      </ul>
    </div>
    <table>
      <thead>
        <tr>
          <th scope="col">User</th>
          <th scope="col">Message</th>
          <th scope="col">Time</th>
        </tr>
      </thead>
      <tbody v-for="item in allWaves">
        <tr>
          <th scope="row">
            <a
              :href="'https://rinkeby.etherscan.io/address/' + item.address"
              target="_blank"
              >{{ item.address }}</a
            >
          </th>
          <td>{{ item.message }}</td>
          <td>{{ item.timeStamp }}</td>
        </tr>
      </tbody>
    </table>
  </div>
</template>
<script>
import { ethers } from "ethers";
import Nprogress from "nprogress";
import "nprogress/nprogress.css";
import { contractAddress, contractABI } from "../utils/constant";
export default {
  data() {
    return {
      message: "Leave a message...",
      currentAccount: null,
      setCurrentAccount: null,
      pagerMessage: null,
      waveCount: 0,
      allWaves: [],
    };
  },
  mounted: function () {
    this.checkIfWalletIsConnected();
    this.setWaveCount();
  },
  methods: {
    checkIfWalletIsConnected: async function () {
      try {
        const { ethereum } = window;
        if (!ethereum) {
          alert("Make sure you have metamask!");
          return;
        } else {
          console.log("We have the ethereum object", ethereum);
        }
        const accounts = await ethereum.request({ method: "eth_accounts" });
        if (accounts.length !== 0) {
          const account = accounts[0];
          console.log("Found an authorized account:", account);
          this.currentAccount = account;
        } else {
          console.log("No authorized account found");
        }
      } catch (error) {
        console.log(error);
      }
    },
    connectWallet: async function () {
      try {
        const { ethereum } = window;
        if (!ethereum) {
          alert("Get MetaMask!");
          return;
        }
        const accounts = await ethereum.request({
          method: "eth_requestAccounts",
        });
        console.log("Connected", accounts[0]);
        this.currentAccount = accounts[0];

        let connectWalletButton = document.querySelector(
          "#vuepress-theme-blog__global-layout > div.content-wrapper > div > div > div > div > ul > li:nth-child(2) > div > button"
        );
        connectWalletButton.disabled = true;
        connectWalletButton.innerText = "Connected";
      } catch (error) {
        console.log(error);
      }
    },
    wave: async function () {
      try {
        const { ethereum } = window;

        if (ethereum) {
          const provider = new ethers.providers.Web3Provider(ethereum);
          const signer = provider.getSigner();
          const wavePortalContract = new ethers.Contract(
            contractAddress,
            contractABI,
            signer
          );

          /*
           * Execute the actual wave from your smart contract
           */
          const waveTxn = await wavePortalContract.wave(this.pagerMessage);
          console.log("Mining...", waveTxn.hash);
          Nprogress.start();
          await waveTxn.wait();
          Nprogress.done();
          console.log("Mined -- ", waveTxn.hash);
        } else {
          console.log("Ethereum object doesn't exist!");
          this.setWaveCount();
        }
      } catch (error) {
        console.log(error);
      }
    },
    setWaveCount: async function () {
      try {
        const { ethereum } = window;

        if (ethereum) {
          const provider = new ethers.providers.Web3Provider(ethereum);
          const signer = provider.getSigner();
          const wavePortalContract = new ethers.Contract(
            contractAddress,
            contractABI,
            signer
          );

          let count = await wavePortalContract.getTotalWaves();
          this.waveCount = count;

          let getWavers = await wavePortalContract.getWaversAndMessage();

          let wavesCleaned = [];
          getWavers.forEach((wave) => {
            wavesCleaned.push({
              address: wave.waverAddress,
              timeStamp: new Date(wave.timeStamp * 1000).toLocaleString(
                "en-US"
              ),
              message: wave.message,
            });
          });

          let reverseWave = wavesCleaned.reverse();
          this.allWaves = reverseWave;

          wavePortalContract.on("NewWave", (from, message, timeStamp) => {
            this.allWaves = [
              {
                address: from,
                timeStamp: new Date(timeStamp * 1000).toLocaleString("en-US"),
                message: message,
              },
              ...this.allWaves,
            ];
          });
        } else {
          console.log("Ethereum object doesn't exist!");
        }
      } catch (error) {
        console.log(error);
      }
    },
  },
};
</script>
<style scoped>
.pagerContainer {
  display: grid;
  justify-content: center;
  justify-items: stretch;
  color: #2c3e50;
  background-color: #fff;
  padding: 20px 32px 20px;
  margin: auto;
  box-shadow: 0 5px 20px rgba(0, 0, 0, 0.03), 0 6px 6px rgba(0, 0, 0, 0.05);
}

.pagerDataContainer {
  display: grid;
}

ul.info {
  display: flex;
  list-style-type: none;
  flex-wrap: wrap;
  justify-content: space-between;
  align-items: center;
  font-size: x-small;
}

.button {
  background-color: rgba(0, 0, 0, 0.644);
  border: none;
  color: #fff;
  text-align: center;
  text-decoration: none;
  display: inline-block;
  cursor: pointer;
}

.primaryButton {
  margin: auto;
}

table {
  table-layout: fixed;
  width: 100%;
  border-collapse: collapse;
  border: 1px solid #000;
  font-size: small;
  border: none;
}

th {
  max-width: 100px;
  white-space: nowrap;
  overflow: hidden;
  text-overflow: ellipsis;
}

thead {
  display: none;
}

thead th:nth-child(1) {
  width: 30%;
}

thead th:nth-child(2) {
  width: 40%;
}

thead th:nth-child(3) {
  width: 25%;
}

th,
td {
  padding: 20px;
}
</style>
