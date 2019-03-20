<template>
  <div>
    <div class="border mb-3 px-3 py-4">
      <h2 @click="selectSeat">name:{{ name }}</h2>
      <p>cabin:{{ cabin }}</p>
      <p>wingRows:{{ wingRows }}</p>
      <p>columns:{{ columns }}</p>
      <button @click="setWingPosition">set wing</button>
      <button @click="setSelectedSeats()">set Seat</button>
    </div>

    <div class="seat-map_container border">
      <template v-for="(row, i) in seats">
        <div class="seat_row" :key="'row' + i">
          <template class="seat_col" v-for="(col, j) in row">
            <div
              v-if="col.type === 'text'"
              :class="`seat-block--text`"
              :key="j"
            >{{ col.content.text }}</div>

            <button
              @click="selectSeat"
              v-if="col.type === 'seat'"
              class="seat-block--seat"
              :class="{ selected: isSelected(col.content.seatNo) }"
              :data-content="JSON.stringify(col.content)"
              :disabled="!col.content.isAvailable"
              :key="j"
            >
              {{
              isSelected(col.content.seatNo)
              ? getPaxIndex(col.content.seatNo)
              : col.content.seatNo.slice(-1)
              }}
            </button>

            <div
              v-if="col.type === 'aisle'"
              :class="`seat-block--aisle`"
              :key="j"
            >{{ col.content.text }}</div>

            <div
              v-if="col.type === 'exit'"
              :class="`seat-block--exit`"
              :key="j"
            >{{ col.content.text }}</div>

            <div
              v-if="col.type === 'facility'"
              :class="`seat-block--facility`"
              :key="j"
            >{{ col.content.text }}</div>
          </template>
        </div>
      </template>

      <div
        ref="wing"
        class="wing"
        :style="{
          top: wingPosition.top + 'px',
          height: wingPosition.height + 'px'
        }"
      ></div>

      <div
        class="seat-wrap"
        v-if="currentSeatInfo"
        ref="seat-wrap"
        :style="{
          top: seatInfoPosition.top + 'px',
          left: seatInfoPosition.left + 'px'
        }"
      >
        <div class="pointer text-center text-danger">▲</div>
        <div class="seat-info card p-3">
          <h4>Seat Info</h4>
          <ul>
            <li>{{ currentSeatInfo.seatNo }}</li>
            <li>{{ currentSeatInfo.seatType }}</li>
            <li>{{ currentSeatInfo.seatInfo.name }}</li>
            <li>{{ currentSeatInfo.seatInfo.description }}</li>
            <li>{{ currentSeatInfo.seatInfo.notification }}</li>
            <li>$:{{ currentSeatInfo.pricing }}</li>
          </ul>
          <select v-model="currPaxIndex">
            <option :value="index" :key="index" v-for="(pax, index) in paxList">
              {{ pax.title }} {{ pax.firstName }}/{{ pax.lastName }} ({{
              selectedSeats[index]
              }})
            </option>
          </select>
          <button @click="sendSelectedSeat(currentSeatInfo.seatNo, currPaxIndex);">ok</button>
          <button @click="resetSeatInfo">cancel</button>
        </div>
      </div>
    </div>
  </div>
</template>
<script>
export default {
  name: "SeatSelector",
  props: {
    seatMapData: {
      type: [Object, Array]
    },

    paxList: {
      type: Array
    }
  },

  data() {
    return {
      flightNo: "JX0110",
      currPaxIndex: null,
      currentSeatInfo: null,
      seatInfoPosition: null,
      wingPosition: {
        top: null,
        width: null
      },
      selectedSeats: null
    };
  },

  methods: {
    selectSeat(evt) {
      this.currentSeatInfo = JSON.parse(
        evt.target.getAttribute("data-content")
      );

      Object.assign(this.currentSeatInfo, {
        seatInfo: this.getSeatTypeInfo(this.currentSeatInfo.seatType)[0]
      });

      this.seatInfoPosition = {
        top: evt.target.offsetTop + evt.target.offsetHeight,
        left:
          evt.target.offsetLeft -
          //小方塊/2
          evt.target.offsetWidth / 2 -
          //info/2 - 小方塊
          (110.5 - evt.target.offsetWidth)
      };

      this.setSelectedSeats(this.currentSeatInfo.seatNo);
    },

    sendSelectedSeat(seatNo, paxIndex) {
      this.currentSeatInfo = null;
      let newPaxList = JSON.parse(JSON.stringify(this.paxList));
      newPaxList[paxIndex].seats.map(item => {
        if (item.flightNo === this.flightNo) {
          item.seatNumber = seatNo;
        }
        return item;
      });
      this.$emit("change", newPaxList);
      setTimeout(() => {
        this.setSelectedSeats();
      }, 0);
    },

    resetSeatInfo() {
      this.currentSeatInfo = null;
    },

    setWingPosition() {
      this.wingPosition = {
        top: 51 * (this.getRowIndex(this.wingRows[0]) + 1),
        height:
          //(翅膀結束 - 翅膀起始 + 1(個數調整))
          (this.getRowIndex(this.wingRows[1]) -
            this.getRowIndex(this.wingRows[0]) +
            1) *
          51 //block size
      };
    },

    getRowIndex(rowNum) {
      return this.seats
        .map(item => item[3].content.text === `${rowNum}`)
        .findIndex(item => item === true);
    },

    getSeatTypeInfo(seatCode) {
      return this.seatTypes.filter(item => item.seatCode === seatCode);
    },

    getSelectedSeats(flightNo) {
      return this.paxList.map(item => {
        if (item.seats.length > 0) {
          return item.seats.filter(item => item.flightNo === flightNo)[0]
            .seatNumber;
        } else {
          return null;
        }
      });
    },

    setSelectedSeats(seatNo) {
      if (seatNo) {
        this.selectedSeats.push(seatNo);
      } else {
        this.selectedSeats = this.getSelectedSeats(this.flightNo);
      }
    },

    isSelected(seatNo) {
      return this.selectedSeats.includes(seatNo);
    },

    getPaxIndex(seatNo) {
      return this.selectedSeats.findIndex(item => item === seatNo) + 1;
    }
  },

  created() {
    this.setSelectedSeats();
    //assign cabin, name, wingrows, columns, seats
    Object.assign(this, this.seatMapData.result.data[0]);
    //assign seatTypes
    Object.assign(this, this.seatMapData.result.meta);
  }
};
</script>
<style scoped>
.seat-map_container {
  position: relative;
  display: inline-block;
  padding-left: 0.2rem;
  background-color: white;
}

.seat_row {
  display: flex;
  justify-content: center;
}

.seat-block--seat {
  width: 3rem;
  height: 3rem;
  background-color: #ccc;
  margin-right: 0.2rem;
  margin-bottom: 0.2rem;
  cursor: pointer;
}

.seat-block--seat:disabled {
  border: 1px solid var(--danger);
}

.seat-block--seat.selected {
  background-color: var(--warning);
  border-radius: 50%;
}

.seat-block--aisle {
  width: 3rem;
  height: 3rem;
  margin-right: 0.2rem;
  margin-bottom: 0.2rem;
  cursor: default;
  background-color: white;
  border: none;
  pointer-events: none;
  text-align: center;
  line-height: 3rem;
}

.seat-block--text {
  width: 3rem;
  height: 3rem;
  margin-right: 0.2rem;
  margin-bottom: 0.2rem;
  cursor: default;
  background-color: white;
  border: none;
  pointer-events: none;
  text-align: center;
  line-height: 3rem;
}

.seat-block--exit {
  width: 3rem;
  height: 3rem;
  margin-right: 0.2rem;
  margin-bottom: 0.2rem;
  cursor: default;
  background-color: white;
  border: none;
  pointer-events: none;
  text-align: center;
  line-height: 3rem;
}

.seat-block--facility {
  width: 3rem;
  height: 3rem;
  margin-right: 0.2rem;
  margin-bottom: 0.2rem;
  cursor: default;
  background-color: white;
  border: none;
  pointer-events: none;
  text-align: center;
  line-height: 3rem;
}

.seat-wrap {
  position: absolute;
  top: 0;
  left: 0;
  z-index: 2;
}

.wing {
  position: absolute;
  top: 100px;
  width: 100%;
  height: 312px;
  background-color: transparent;
  pointer-events: none;
}

.wing:before {
  content: "";
  display: block;
  position: absolute;
  width: 150px;
  height: 100%;
  left: -154px;
  top: 0;
  background: linear-gradient(244deg, #fff, rgba(255, 255, 255, 0));
  transform: skewY(-34deg);
  z-index: 1;
}

.wing:after {
  content: "";
  display: block;
  position: absolute;
  width: 150px;
  height: 100%;
  right: -148px;
  top: 0;
  background: linear-gradient(122deg, #fff, rgba(255, 255, 255, 0));
  transform: skewY(34deg);
  z-index: 1;
}
</style>
