<template>
  <div class="bg-white p-4">
    <div class="flex flex-col sm:flex-row justify-between text-black items-center w-8/12 mx-auto mt-12">
      <div class="mx-auto flex items-center">
        <w3m-core-button></w3m-core-button>
      </div>
    </div>

    <div class="flex flex-col flex-wrap justify-between text-black w-8/12 mx-auto mt-4">
      <div class="flex flex-col justify-start font-bold text-2xl mx-auto">
        <div class="mt-4">
          Quantity
          <input class="w-12 text-center border-2 border-gray-400" type="number" v-model="quantity" />
        </div>

      </div>

      <div class="flex flex-col justify-between font-bold text-xl lg:text-2xl mx-auto">
        <div>
          <div class="mx-auto">
            <div class="text-lg text-gray-400 font-normal text-center mt-4 mb-3">
          Each NFT is .3 BNB
        </div>
            <div class="mx-auto flex justify-center" v-if="quantitywithprice <= bnbAmount">
              <button @click="handleClick" class="card">
                MINT NFT <br>
                {{ formattedPrice }}
              </button>
            </div>
            <div v-else>
              <button @click="handleClick" class="card">
                MINT NFT <br>
                {{ formattedPrice }}
              </button>
              <!-- Display a message or alternative content when the condition is not met -->
              <p v-if="!this.account" class="text-center text-[red]">Please connect your wallet to Mint</p>
              <p v-else class="text-center text-[red]">Insufficient BNB balance</p>
            </div>

            <div class="text-center text-2xl mt-8">INCREDIBULLS MINTED</div>
            <div class="text-center text-[#fd5a08]">
             {{ nftCount }}/{{ maxTokenID }}
            </div>
          </div>
        </div>
      </div>
    </div>

    <div v-if="showModal">
      <div>
        <div class="w-screen h-screen bg-black absolute top-0 opacity-60"></div>
        <div class="absolute w-screen h-screen top-0">
          <div class="grid place-items-center h-screen mx-auto">
            <div class="p-4 bg-white opacity-100 rounded-xl">
              <div class="text-black font-bold text-2xl flex justify-end">
                <button @click="handleCloseModal">X</button>
              </div>
              <div class="sm:px-20">
                <div>
                  <h5 class="sm:text-7xl text-2xl text-center text-[#fd5a08] font-extrabold">
                    INCREDIBULLS
                  </h5>
                </div>
                <div class="flex flex-col justify-between mx-6 mt-6">
                  <div class="flex flex-row justify-between text-black font-bold sm:text-2xl">
                    <p>Name</p>
                    <p>The incredibulls</p>
                  </div>
                  <div class="flex flex-row justify-between text-black font-bold sm:text-2xl">
                    <p>TokenID</p>
                    <p>{{ nftAmount }}</p>
                  </div>
                </div>
                <div>
                  <img class="mx-auto" src="https://steakdincredibulls.com/images/mystery.png" alt="" />
                </div>
              </div>
            </div>
          </div>
        </div>
      </div>
    </div>


    <div v-if="showModalBeginning">
      <div>
        <div class="w-screen h-screen bg-black absolute top-0 opacity-60"></div>
        <div class="absolute w-screen h-screen top-0">
          <div class="grid place-items-center h-screen mx-auto">
            <div class="p-4 bg-white opacity-100 rounded-xl">
              <div class="text-black font-bold text-2xl flex justify-end">
                <button @click="handleCloseModalBeginning">X</button>
              </div>
              <div class="sm:px-20">
                <img src="https://steakdincredibulls.com/images/Home_3.png" alt="" srcset="">
              </div>
            </div>
          </div>
        </div>
      </div>
    </div>
  </div>
</template>

<script>
/* wallet connect config */
import { EthereumClient, w3mConnectors, w3mProvider } from '@web3modal/ethereum'
import { Web3Modal } from '@web3modal/html'
import { configureChains, createConfig, prepareWriteContract, writeContract, readContract, getAccount, fetchBalance } from '@wagmi/core'

import { bsc } from '@wagmi/core/chains'

const chains = [bsc]
const projectId = '04dfd93692ffd288ae5abfcc4d3053eb'

const { publicClient } = configureChains(chains, [w3mProvider({ projectId })])
const wagmiConfig = createConfig({
  autoConnect: true,
  connectors: w3mConnectors({ projectId, version: 1, chains }),
  publicClient
})
const ethereumClient = new EthereumClient(wagmiConfig, chains)
const web3modal = new Web3Modal({ projectId }, ethereumClient)

import { nftABI } from "./nftABI.js"
import { parseGwei } from 'viem'
const bullAddress = "0x6489265Bf18185cA693017E7448bD77E8F80A524"




export default {



  data() {
    return {
      bullAddress: '0x6489265Bf18185cA693017E7448bD77E8F80A524',
      connectedAddress: null,
      showModal: false,
      showModalBeginning: false,
      showModalConnect: false,
      quantity: 0,
      price: 0,
      count: null,
      data: null,
      nftCount: 0,
      maxTokenID: 0,
      nftAmount: 0,
      bnbAmount: 0,
      quantitywithprice: 0,
      account: '',
    };
  },
  mounted() {
    this.getNFTInfo();
    this.getAccountInfo();
  },
  watch: {
    quantitywithprice(newValue, oldValue) {
      console.log('quantitywithprice changed:', newValue);
      this.updateShowMintButton();
    },
    bnbAmount(newValue, oldValue) {
      console.log('bnbAmount changed:', newValue);
      this.updateShowMintButton();
    },
    quantity(newValue, oldValue) {
      // This will be called whenever `quantity` changes
      console.log('quantity changed:', newValue);
      // Perform any desired action here, such as updating the price based on the new quantity
      this.updatePrice();
    },
  },
  computed: {
    showMintButton() {
      console.log(this.quantitywithprice)
      return this.quantitywithprice <= this.bnbAmount;
    },
    formattedPrice() {
      if (this.price === null) {
        return '';
      }
      if (this.price.toString() === '0') {
        return '(0.00 BNB)';
      }
      return `(${(this.price.toString() * this.quantity) / Math.pow(10, 18)} BNB)`;
    },
  },
  methods: {
    async getAccountInfo() {
      let curAccount = getAccount();
      this.account = curAccount.address;
      const balance = await fetchBalance({
        address: curAccount.address,
      })
      this.bnbAmount = balance.formatted;
      console.log(balance);
    },
    async getNFTInfo() {
      this.nftCount = await readContract({
        address: bullAddress,
        abi: nftABI,
        functionName: 'nftCount'
      })

      this.maxTokenID = await readContract({
        address: bullAddress,
        abi: nftABI,
        functionName: 'maxTokenID'
      })

      this.decrementValue()
      this.nftAmount = this.nftCount
    },

    decrementValue() {
      this.nftCount = this.nftCount.toString() - 1;
    },

    updateShowMintButton() {
      console.log('Updating showMintButton');
      this.$forceUpdate();
    },

    async updatePrice() {
      this.price = await readContract({
        address: bullAddress,
        abi: nftABI,
        functionName: 'mintPrice',
        // You can pass the updated quantity as an argument if necessary
        arguments: [this.quantity],
      });
      this.quantitywithprice = (this.price.toString() * this.quantity) / Math.pow(10, 18);

    },
    setQuantity(value) {
      this.quantity = value;
    },

    handleLogout() {
      // Logout logic
    },
    handleOpenModalconnect() {
      // Open modal connect logic
    },
    setQuantity(value) {
      this.quantity = value;
    },
    async handleClick() {
      try {
        let account = getAccount()
        console.log((this.price.toString() * this.quantity) / Math.pow(10, 18))
        const price = Number(this.price); // Convert the string to a number
        const quantity = Number(this.quantity); // Convert the string to a number
        const { hash } = await writeContract({
          address: this.bullAddress,
          abi: nftABI,
          functionName: 'mintNFT',
          args: [account.address, this.quantity],
          value: (price * quantity).toString(),
        })
        // Contract execution succeeded, open the modal
        this.nftAmount = this.nftCount + this.quantity
        this.showModal = true;
      } catch (error) {
        // Handle any error that occurred during contract execution
        console.error(error);
      }




    },
    handleCloseModal() {
      this.showModal = false;
    },

    handleCloseModalBeginning() {
      this.showModalBeginning = false;
    },

    handleCloseModalconnect() {
      this.showModalConnect = false;
    },
  },
};
</script>

<style scoped>
.container {
  padding: 0 2rem;
}

.main {
  min-height: 100vh;
  padding: 4rem 0;
  flex: 1;
  display: flex;
  flex-direction: column;
  justify-content: center;
  align-items: center;
}

.title a {
  color: #f213a4;
  text-decoration: none;
}

.title a:hover,
.title a:focus,
.title a:active {
  text-decoration: underline;
}

.title {
  margin: 0;
  line-height: 1.15;
  font-size: 4rem;
}

.title,
.description {
  text-align: center;
}

.connect {
  margin-bottom: 2rem;
}

.description {
  margin-top: 3rem;
  margin-bottom: 2rem;
  line-height: 1.5;
  font-size: 1.5rem;
}

.code {
  background: #fafafa;
  border-radius: 5px;
  padding: 0.75rem;
  font-size: 1.1rem;
  font-family: Menlo, Monaco, Lucida Console, Liberation Mono, DejaVu Sans Mono,
    Bitstream Vera Sans Mono, Courier New, monospace;
}

.grid {
  display: flex;
  align-items: center;
  justify-content: center;
  flex-wrap: wrap;
  max-width: 1200px;
}

.card {
  background-color: #fd5a08 !important;
  color: white !important;
  width: 375px !important;
  height: 100px !important;
  font-weight: 700 !important;
  font-size: 1.5em !important;
  border-radius: 20px;
  margin-bottom: 0px;

}

.card:hover,
.card:focus,
.card:active {
  background-color: #272c34;
  color: #0ea5e9;
  border-color: #0ea5e9;
}

.card h2 {}

.card p {}

.logo {
  height: 1em;
  margin-left: 0.5rem;
}

@media (max-width: 600px) {
  .grid {
    width: 100%;
    flex-direction: column;
  }
}

.card,
.footer {
  border-color: #222;
}

.code {
  background: #111;
}

.logo img {
  filter: invert(1);
}


@media screen and (max-width: 1025px) {
  .card {
    background-color: #fd5a08 !important;
    color: white !important;
    width: 275px !important;
    height: 100px !important;
    font-weight: 700 !important;
    font-size: 1.3em !important;

    border-radius: 10px;

  }
}

@media screen and (max-width: 426px) {
  .card {
    background-color: #fd5a08 !important;
    color: white !important;
    width: 275px !important;
    height: 100px !important;
    font-weight: 700 !important;
    font-size: 1.2em !important;

    margin-bottom: 30px;
    border-radius: 10px;

  }
}

@media screen and (max-width: 321px) {
  .card {
    background-color: #fd5a08 !important;
    color: white !important;
    width: 175px !important;
    height: 100px !important;
    font-weight: 700 !important;
    font-size: 1.2em !important;

    border-radius: 10px;
    margin-bottom: 30px;
  }
}

.tw-connect-wallet {
  background: #69C1F6 !important;
}

.tw-connected-wallet {
  background: #69C1F6 !important;

}
</style>