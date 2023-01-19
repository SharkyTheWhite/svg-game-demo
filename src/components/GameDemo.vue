<template>
  <div>
    <div v-for="pad in pads" :key="pad.gamepad.id">
      <b>[{{ pad.gamepad.index }}] {{ pad.gamepad.id }}</b>
      <br />
      <svg width="512px" height="512px" viewBox="0 0 512 512">
        <circle cx="256" cy="256" r="200" style="fill: gray"></circle>
        <circle
          :cx="256"
          :cy="256"
          r="25"
          style="fill: transparent; stroke: green"
        ></circle>
        <circle
          :cx="256 + pad.axes[2] * 200"
          :cy="256 + pad.axes[3] * 200"
          r="30"
          style="fill: transparent; stroke: blue"
        ></circle>
        <circle
          :cx="256 + pad.axes[0] * 200"
          :cy="256 + pad.axes[1] * 200"
          r="20"
          style="fill: black"
        ></circle>
      </svg>
    </div>
    <div v-if="pads.length === 0">Please tap your gamepad.</div>
  </div>
</template>

<script setup lang="ts">
import { ref } from 'vue'

interface RumbleActuator extends GamepadHapticActuator {
  // eslint-disable-next-line  @typescript-eslint/no-explicit-any
  playEffect(type: string, params: any): Promise<boolean>
}

interface ExtraGamepad extends Gamepad {
  vibrationActuator?: RumbleActuator
}

class GamepadWrapper {
  public gamepad: Gamepad
  public axes: number[]

  public constructor(gamepad: Gamepad) {
    this.gamepad = gamepad
    this.axes = Array<number>(gamepad.axes.length)
  }

  public refresh() {
    const gp = (navigator.getGamepads()[this.gamepad.index] ??
      this.gamepad) as ExtraGamepad
    for (let i = 0; i < this.axes.length; i++) this.axes[i] = gp.axes[i]
    if (this.axes[0] > 0.5 && gp.vibrationActuator) {
      gp.vibrationActuator.playEffect('dual-rumble', {
        startDelay: 0,
        duration: 200,
        weakMagnitude: 1.0,
        strongMagnitude: 1.0,
      })
    }
  }
}

//const pads = ref<Gamepad[]>([])
const pads = ref<GamepadWrapper[]>([])

window.addEventListener('gamepadconnected', (e) => {
  console.log(
    'Gamepad connected at index %d: %s. %d buttons, %d axes.',
    e.gamepad.index,
    e.gamepad.id,
    e.gamepad.buttons.length,
    e.gamepad.axes.length
  )
  pads.value.push(new GamepadWrapper(e.gamepad))
})

window.addEventListener('gamepaddisconnected', (e) => {
  console.log(
    'Gamepad disconnected from index %d: %s',
    e.gamepad.index,
    e.gamepad.id
  )
})

const frameUpdate = () => {
  pads.value.forEach((pad) => pad.refresh())
}

//setInterval(frameUpdate,20)
let clog = 20
const aniFrame = (time?: DOMHighResTimeStamp) => {
  frameUpdate()
  if (clog > 0) {
    clog--
    console.log({ aniTimeStamp: time })
  }
  requestAnimationFrame(aniFrame)
}

aniFrame()
</script>

<style scoped></style>
