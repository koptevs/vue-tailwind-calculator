<script setup>
import { evaluate } from "mathjs";
import { gsap } from "gsap";
</script>

<template>
  <main>
    <div class="md:grid md:grid-cols-5 gap-2 max-w-4xl min-w-max">
      <div class="md:col-span-3">
        <div
          style="min-width: 500px"
          class="h-28 bg-sky-900 text-slate-100 text-4xl flex p-6 font-bold mb-2 opacity-90 items-center"
        >
          {{ res }}
        </div>
        <div class="grid grid-cols-4 p-4 gap-2 bg-sky-900 opacity-95">
          <div
            @click="onButtonClearClick"
            title="Clear screen"
            :class="[isOutputEmpty() ? 'hidden' : '']"
            class="min-w-140 bg-sky-800 bg-opacity-90 active:bg-sky-900 text-slate-200 active:text-white font-bold text-6xl w-full h-22 p-4 xl:h-24 flex items-center justify-center rounded-sm shadow-xl active:shadow-none transition-all cursor-pointer"
          >
            C
          </div>
          <div
            @click="onButtonAllClearClick"
            title="Clear screen"
            :class="[isOutputEmpty() ? '' : 'hidden']"
            class="min-w-140 bg-sky-800 bg-opacity-90 active:bg-sky-900 text-slate-200 active:text-white font-bold text-5xl w-full h-22 p-4 xl:h-24 flex items-center justify-center rounded-sm shadow-xl active:shadow-none transition-all cursor-pointer"
          >
            AC
          </div>
          <div
            @click="removeLastSymbol"
            class="bg-sky-800 bg-opacity-90 active:bg-sky-900 text-slate-200 active:text-white font-bold text-6xl w-full h-22 p-4 xl:h-24 flex items-center justify-center rounded-sm shadow-xl active:shadow-none transition-all cursor-pointer"
          >
            &larr;
          </div>
          <div
            v-for="(button, value) in buttons"
            @click="onButtonNumericClick(button.value)"
            :key="button.id"
            :title="button.title"
            class="bg-sky-800 bg-opacity-90 active:bg-sky-900 text-slate-200 active:text-white font-bold text-6xl w-full h-22 p-4 xl:h-24 flex items-center justify-center rounded-sm shadow-xl active:shadow-none transition-all cursor-pointer"
          >
            {{ button.label }}
          </div>
          <div
            @click="onButtonEqualsClick"
            class="bg-sky-800 bg-opacity-90 active:bg-sky-900 text-slate-200 active:text-white font-bold text-6xl w-full h-22 p-4 xl:h-24 flex items-center justify-center rounded-sm shadow-xl active:shadow-none transition-all cursor-pointer"
          >
            =
          </div>
        </div>
      </div>
      <div class="bg-slate-600 md:col-span-2 p-4 opacity-70 text-4xl text-white">
        <div class="history-header bg-slate-800 p-2">
          <h2>History</h2>
          <p class="text-lg">click on expression to load.</p>
          <p>{{ operationCompleted ? "Completed" : "Not completed" }}</p>
        </div>
        <div
          class="cursor-pointer"
          v-for="record in history"
          @click="onHistoryItemClick(record)"
        >
          {{ record }}
        </div>
      </div>
    </div>
  </main>
</template>

<script>
export default {
  data() {
    return {
      history: [],
      operationStarted: false,
      operationCompleted: false,
      count: 0,
      precision: 5,
      res: "",
      buttons: [
        { id: "modulo", value: "%", label: "%", title: "" },
        { id: "divide", value: "/", label: "/", title: "" },
        { id: "seven", value: "7", label: "7", title: "" },
        { id: "eight", value: "8", label: "8", title: "" },
        { id: "nine", value: "9", label: "9", title: "" },
        { id: "multiply", value: "*", label: "x", title: "" },
        { id: "four", value: "4", label: "4", title: "" },
        { id: "five", value: "5", label: "5", title: "" },
        { id: "six", value: "6", label: "6", title: "" },
        { id: "subtract", value: "-", label: "-", title: "" },
        { id: "one", value: "1", label: "1", title: "" },
        { id: "two", value: "2", label: "2", title: "" },
        { id: "three", value: "3", label: "3", title: "" },
        { id: "add", value: "+", label: "+", title: "" },
        { id: "cleanHistory", value: "", label: "", title: "" },
        { id: "zero", value: "0", label: "0", title: "" },
        { id: "comma", value: ".", label: ",", title: "" },
      ],
    };
  },
  methods: {
    onButtonNumericClick(e) {
      if (this.isOperator(e) && (this.isOutputEmpty() || this.operationCompleted)) return;
      if (this.isLastSymbolOperator() && this.isOperator(e)) return;
      if (this.operationCompleted) {
        // this.addToHistory(this.res);
        this.clearOutput();
        this.operationCompleted = false;
      }
      this.res += e;
    },
    onButtonClearClick() {
      this.clearOutput();
      this.operationCompleted = true;
    },
    onButtonAllClearClick() {
      this.clearOutput();
      this.clearHistory();
    },
    onHistoryItemClick(arg) {
      this.operationCompleted = false;
      this.res = arg.split("=")[0].trim();
    },

    onButtonEqualsClick() {
      if (this.operationCompleted) return;
      if (!this.isOutputEmpty() && !this.isLastSymbolOperator()) {
        this.res +=
          " = " +
          Math.round(evaluate(this.res) * Math.pow(10, this.precision)) /
            Math.pow(10, this.precision);
        this.addToHistory(this.res);
        // this.clearOutput()
        this.operationCompleted = true;
      }
    },
    removeLastSymbol() {
      if (!this.operationCompleted) {
        this.res = this.res.slice(0, -1);
      }
    },
    clearAll() {
      if (this.operationCompleted) addToHistory(this.res);
      this.res = "";
    },
    // Helper functions
    isOperator(el) {
      return ["+", "-", "*", "/", "%"].includes(el);
    },
    clearHistory() {
      this.history = [];
    },
    addToHistory(el) {
      this.history.unshift(el);
    },
    clearOutput() {
      this.res = "";
    },
    isOutputEmpty() {
      return !Boolean(this.res.length);
    },
    isLastSymbolOperator() {
      return this.isOperator(this.res.trim().slice(-1));
    },
  },
};
</script>

<style></style>
