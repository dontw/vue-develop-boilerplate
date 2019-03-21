<template>
  <div id="app">
    <div class="container">
      <h1>seat map</h1>
      <div class="border mb-3">
        <p>flightInfo:</p>
        <template v-for="(flight, index) in flightInfo">
          <p :key="'flight' + index">{{ flight }}</p>
        </template>
        <p>paxList:</p>
        <template v-for="(pax, index) in paxList">
          <p :key="index">{{ pax }}</p>
        </template>
      </div>
      <div class="border text-center bg-secondary">
        <h3>去程</h3>
        <button @click="setPaxSeats">Add seat</button>
        <button @click="removePaxSeats">Remove Seat</button>
        <SeatSelector :seatMapData="seatMapData" :paxList="paxList" @change="changePaxSeats"/>
      </div>
    </div>
  </div>
</template>

<script>
import seatMapData from "./seatMap1.js";
import SeatSelector from "@/components/SeatSelector.vue";
export default {
  name: "App",
  components: { SeatSelector },
  data() {
    return {
      seatMapData: null,
      paxList: [
        {
          id: "SKH-1-EXT",
          paxType: "ADT",
          title: "MR",
          lastName: "DOGE",
          firstName: "SHIBA",
          memberCard: null,
          noFirstName: false,
          hasFirstName: true,
          seats: []
        },
        {
          id: "SKH-2-EXT",
          paxType: "ADT",
          title: "MS",
          lastName: "CATE",
          firstName: "MEWO",
          memberCard: null,
          noFirstName: false,
          hasFirstName: true,
          seats: []
        },
        {
          id: "SKH-3-EXT",
          paxType: "CHD",
          title: "PRINCE",
          lastName: "MARU",
          firstName: "TARO",
          memberCard: null,
          noFirstName: false,
          hasFirstName: true,
          seats: []
        }
      ],
      flightInfo: [
        {
          date: "2019/04/02",
          depTime: "08:00",
          arrTime: "09:55",
          depAirport: "TPE",
          depTerminal: "1",
          arrAirport: "MFM",
          arrTerminal: null,
          flightNo: "JX0110",
          flightId: "EAI-2-OFF",
          aircraft: "321",
          duration: 6900
        },
        {
          date: "2019/04/04",
          depTime: "11:30",
          arrTime: "13:20",
          depAirport: "MFM",
          depTerminal: null,
          arrAirport: "TPE",
          arrTerminal: "1",
          flightNo: "JX0111",
          flightId: "EAI-3-OFF",
          aircraft: "321",
          duration: 6600
        }
      ],
      cartInfo: {
        flightInformation: [
          {
            date: "2019/04/02",
            depTime: "08:00",
            arrTime: "09:55",
            depAirport: "TPE",
            depTerminal: "1",
            arrAirport: "MFM",
            arrTerminal: null,
            flightNo: "JX0110",
            flightId: "EAI-2-OFF",
            aircraft: "321",
            duration: 6900
          },
          {
            date: "2019/04/04",
            depTime: "11:30",
            arrTime: "13:20",
            depAirport: "MFM",
            depTerminal: null,
            arrAirport: "TPE",
            arrTerminal: "1",
            flightNo: "JX0111",
            flightId: "EAI-3-OFF",
            aircraft: "321",
            duration: 6600
          }
        ],
        travelers: [
          {
            paxType: "ADT",
            title: null,
            firstName: null,
            lastName: null,
            birthDay: null,
            id: "SKH-4-EXT",
            seats: []
          }
        ],
        contact: [],
        unitPrice: [
          {
            travelerId: "SKH-4-EXT",
            prices: {
              base: 7750,
              fee: 0,
              tax: [
                { type: "TWAE", price: 500 },
                { type: "MOSE", price: 115 },
                { type: "WNDP", price: 421 }
              ]
            },
            totalPrice: 8786
          }
        ]
      }
    };
  },

  methods: {
    changePaxSeats(newVal) {
      this.paxList = newVal;
    },

    setPaxSeats() {
      this.paxList[0].seats = [
        {
          flightNo: "JX0110",
          seatNumber: "5D"
        },
        {
          flightNo: "JX0111",
          seatNumber: "5C"
        }
      ];
      this.paxList[1].seats = [
        {
          flightNo: "JX0110",
          seatNumber: "6D"
        },
        {
          flightNo: "JX0111",
          seatNumber: "6C"
        }
      ];

      this.paxList[2].seats = [
        {
          flightNo: "JX0110",
          seatNumber: "7D"
        },
        {
          flightNo: "JX0111",
          seatNumber: "7C"
        }
      ];
    },

    removePaxSeats() {
      this.paxList[0].seats = [];
      this.paxList[1].seats = [];
    }
  },

  created() {
    this.seatMapData = seatMapData;
  }
};
</script>
