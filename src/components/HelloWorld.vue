<template>
  <div class="relative">
    <div :key="sent">sent: {{ sent }}</div>
    <div>stack : {{ Math.max(0, count - sent) }}</div>
    <button ref="button">click Me!</button>
    <div class="animation absolute left-0 bottom-0">
      <div>{{ count }}</div>
      <div class="bar" :style="{ height: `${count * 3}px` }"></div>
    </div>
  </div>
</template>

<script>
import { fromEvent, interval, merge } from "rxjs";
import {
  switchMap,
  takeUntil,
  takeWhile,
  tap,
  finalize,
  mapTo,
} from "rxjs/operators";

const config = {
  UPDATE_TIME: 100,
  COMBO_TIME: 3 * 1000,
  SEND_TIME: 0.5 * 1000,
};
export default {
  name: "HelloWorld",
  props: {
    msg: String,
  },

  data() {
    return {
      sent: 0,
      count: 0,
      MAX_COUNT: config.COMBO_TIME / config.UPDATE_TIME,
    };
  },

  mounted() {
    const press = fromEvent(this.$refs.button, "pointerdown");
    const up = fromEvent(this.$refs.button, "pointerup");

    const send = press.pipe(
      switchMap(() =>
        interval(config.SEND_TIME)
          .pipe(takeUntil(recycle))
          .pipe(tap(() => this.send()))
      )
    );

    const bubble = press.pipe(
      switchMap(() =>
        interval(config.UPDATE_TIME)
          .pipe(
            takeWhile(() => this.count < this.MAX_COUNT),
            takeUntil(recycle),
            mapTo(1)
          )
          .pipe(tap((v) => (this.count += v)))
      )
    );

    const recycle = up.pipe(
      switchMap(() =>
        interval(config.UPDATE_TIME)
          .pipe(
            takeWhile(() => this.count > 0),
            takeUntil(bubble),
            mapTo(-1)
          )
          .pipe(
            tap((v) => {
              this.count += v;
            }),
            finalize(() => {
              this.sent = this.count;
              if (this.count === 0) {
                this.reset();
              }
            })
          )
      )
    );

    merge(bubble, recycle, send).subscribe();
  },

  methods: {
    send() {
      this.sent = Math.max(0, this.count - this.sent);
    },
    reset() {
      this.sent = 0;
      this.count = 0;
      this.level = 0;
    },
  },
};
</script>

<style lang="scss" scoped>
.bar {
  width: 60px;
  height: 120px;
  background: linear-gradient(#e66465, #9198e5);
}
</style>
