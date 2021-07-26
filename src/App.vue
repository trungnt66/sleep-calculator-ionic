<template>
  <ion-app id="app">
    <!-- <ion-router-outlet /> -->
    <!-- <div>hello</div> -->
    <h1 class="text-center">SLEEP CALCULATOR</h1>
    <h4 class="text-center">If I want to wake up at:</h4>
    <input type="time" v-model="timeText" @change="timeTextChange" />
    <button class="btn" @click="calculateGoToBed">
      When should I go to bed?
    </button>
    <h3 class="text-center">or</h3>
    <h4 class="text-center">
      If I'm sleepy right now, so my alarm should be set for…
    </h4>
    <button class="btn mb-1" @click="calculateSetAlarm">
      Click to find out
    </button>

    <div v-show="listCycle.length">
      <h4 v-if="purpose === purposes.getTimeSleep">You should go to bed at:</h4>
      <h4 v-else>You should set alarm at:</h4>
    </div>
    <transition-group name="list" tag="div" class="cycle-wrapper" mode="in-out">
      <div
        class="cycle"
        v-for="el in listCycle"
        v-bind:key="el"
        :class="{ 'cycle-recommended': el.recommended }"
      >
        {{ el.value }}
      </div>
      <!-- </div> -->
    </transition-group>
  </ion-app>
</template>

<script lang="ts">
import { IonApp } from "@ionic/vue";
import { defineComponent, ref } from "vue";

const appTimeManager = (function () {
  const time = new Date();
  const settime = (timeString: string) => {
    if (timeString && typeof timeString === "string") {
      const [hour, minute] = timeString.split(":");
      if (hour && minute) {
        time.setHours(Number(hour));
        time.setMinutes(Number(minute));
        time.setSeconds(0);
      }
    }
  };

  const timeValue = () => {
    let hours = time.getHours();
    const minutes = time.getMinutes();
    const ampm = hours >= 12 ? "PM" : "AM";
    hours = hours % 12;
    hours = hours ? hours : 12; // the hour '0' should be '12'
    const minutesstr = minutes < 10 ? "0" + minutes : minutes;
    const strTime = hours + ":" + minutesstr + " " + ampm;
    return strTime;
  };

  const addtime = (timeString: any, minute: any) => {
    settime(timeString);
    time.setMinutes(time.getMinutes() + minute);
    return timeValue();
  };

  const getListCytle = (timeString: any, isCalGoToBed: any) => {
    if (!isCalGoToBed) {
      const currentTime = new Date();
      timeString = currentTime.getHours() + ":" + currentTime.getMinutes();
    }

    const listCycle = [];
    for (let index = 3; index < 9; index++) {
      // 90 miniute is one cycle 15 minute is average time to fall as sleep
      listCycle.push({
        value: addtime(
          timeString,
          90 * index * (isCalGoToBed ? -1 : 1) + (isCalGoToBed ? -15 : +15)
        ),
        recommended: index === 5 || index === 6,
      });
    }

    return listCycle.reverse();
  };

  return {
    settime,
    addtime,
    getListCytle,
    get time() {
      return timeValue();
    },
  };
})();

export default defineComponent({
  name: "App",
  setup() {
    const purposes = {
      getTimeSleep: 1,
      getAlarm: 2,
    };
    const timeText = ref("07:00");
    const listCycle = ref([]);

    const purpose = ref(0);
    console.log(appTimeManager.time);

    const timeTextChange = () => {
      listCycle.value = [];
    };

    const calculateGoToBed = () => {
      listCycle.value = appTimeManager.getListCytle(timeText.value, true) as any;
      purpose.value = purposes.getTimeSleep;
    };

    const calculateSetAlarm = () => {
      listCycle.value = appTimeManager.getListCytle(timeText.value, false) as any;
      purpose.value = purposes.getAlarm;
    };
    return {
      timeText,
      listCycle,
      purpose,
      purposes,
      calculateGoToBed,
      calculateSetAlarm,
      timeTextChange,
    };
  },
  components: {
    IonApp,
  },
});
</script>

<style>
:root {
    /* color */
    --light: #dbedf3;
    --dark: #404b69;
    --darker: #283149;
    --primary: #00818a;
}

html {
    font-size: 18px;
}

body {
    padding: 0;
    margin: 0;
}

/* margin - padding */
.mb-1 {
    margin-bottom: 1rem;
}
/* end margin-padding */
/* text */
.text-center {
    text-align: center;
}
/* end text */

/* button */
.btn {
    cursor: pointer;
    color: var(--light);
    /* border: 2px solid var(--light); */
    border: none;
    box-shadow: 0 0 1.5rem 0 var(--darker);
    padding: 0.5rem 1rem;
    border-radius: 5rem;
    box-sizing: border-box;
    transition: all 0.3s ease-in-out;
    position: relative;
    z-index: 0;
    overflow: hidden;
    background: transparent;
    font-size: 1rem;
}

.btn::before {
    content: "";
    position: absolute;
    top: 0;
    left: 0;
    height: 100%;
    width: 100%;
    z-index: -1;
    background: var(--light);
    transform: translateX(-101%);
    transition: transform 500ms ease-in-out;
    border-radius: 5rem;
}

.btn:hover::before {
    transform: translateX(0);
}

.btn:hover {
    transform: scale(1.1);
    /* border: 2px solid transparent; */
    color: var(--darker) !important;
}
.btn:focus::before {
    transform: translateX(0);
}

.btn:focus {
    transform: scale(1.1);
    /* border: 2px solid transparent; */
    color: var(--darker) !important;
    outline:0;
}

/* button-link */
.btn.btn-link {
    /* border: 0 solid transparent; */
}
.btn.btn-link:hover {
    transform: scale(1);
}
/* end button-link */
/* end button */

/* start time picker */
input[type="time"] {
    border: none;
    padding: 0.5rem;
    font-size: 20px;
    border-radius: 3rem;
    margin-bottom: 1rem;
    color: var(--darker);
    background: var(--light);
    outline: none;
}
/* end time picker */

#app {
  font-family: Avenir, Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  background: var(--dark);
  min-height: 100vh;
  width: 100vw;
  display: flex;
  align-items: center;
  flex-direction: column;
  color: var(--light);
  overflow: auto;
  position: relative;
  padding: 1rem;
  box-sizing: border-box;
  justify-content: unset;
}

#app .cycle {
  margin: 0.5rem;
  padding: 0.5rem;
  width: 7rem;
  background: var(--darker);
  color: var(--light);
  text-align: center;
  position: relative;
  overflow: hidden;
}

#app .cycle.cycle-recommended {
  background: var(--light);
  color: var(--darker);
}

#app .cycle.cycle-recommended::after {
  content: "👍";
  position: absolute;
  right: 0;
  top: 0;
  color: red;
  background: orange;
  padding: 0 2px;
  height: 100%;
  display: flex;
  align-items: center;
  box-sizing: border-box;
  font-size: 0.8rem;
  transition: all 0.5s;
  transition-delay: 0.5s;
}

#app .cycle-container > div:first {
  margin-top: 2rem;
}

#app .cycle-container {
  max-height: 0rem;
  overflow: hidden;
  transition: all 2s ease;
}

#app .cycle-container.show-cycle {
  max-height: 100rem;
}

#app .list-enter-active {
  transition: all 0.3s ease-out;
  transition-delay: 0.3s;
}
#app .list-leave-active {
  transition: all 0.3s ease-in;
}

#app .list-enter-from {
  opacity: 0;
  height: 0;
  padding: 0;
  overflow: hidden;
  transform: scaleY(0);
}
#app .list-leave-to {
  opacity: 0;
  /* height: 0; */
  transform: translateX(5rem);
}

#app .list-enter-from::after {
  transform: translateX(101%);
}

#app .cycle-wrapper {
  display: grid;
  grid-template-columns: auto auto;
}

@media screen and (max-width: 1200px) {
  #app .cycle-wrapper {
    grid-template-columns: auto auto;
  }
}
</style>
