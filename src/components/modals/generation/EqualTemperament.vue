<script setup lang="ts">
import { DEFAULT_NUMBER_OF_COMPONENTS } from "@/constants";
import Scale from "@/scale";
import { computed, ref, watch } from "vue";
import Modal from "@/components/ModalDialog.vue";
import { Interval } from "@/interval";
import ExtendedMonzo from "@/monzo";
import ScaleLineInput from "@/components/ScaleLineInput.vue";
import { splitText } from "@/components/modals/tempering-state";

const emit = defineEmits(["update:scale", "update:scaleName", "cancel"]);

const octave = new Interval(
  ExtendedMonzo.fromNumber(2, DEFAULT_NUMBER_OF_COMPONENTS),
  "ratio"
);

const divisions = ref(5);
const equaveString = ref("2/1");
const equave = ref(octave);

const jumpsString = ref("1 1 1 1 1");
const degreesString = ref("1 2 3 4 5");

const jumpsElement = ref<HTMLTextAreaElement | null>(null);
const degreesElement = ref<HTMLTextAreaElement | null>(null);

const jumps = computed(() =>
  splitText(jumpsString.value).map((token) => parseInt(token))
);
const degrees = computed(() =>
  splitText(degreesString.value).map((token) => parseInt(token))
);

watch(jumps, (newValue) => {
  if (newValue.includes(NaN)) {
    jumpsElement.value!.setCustomValidity("Step size not an integer");
  } else {
    jumpsElement.value!.setCustomValidity("");
  }
});

watch(degrees, (newValue) => {
  if (newValue.includes(NaN)) {
    degreesElement.value!.setCustomValidity("Degree not an integer");
  } else {
    degreesElement.value!.setCustomValidity("");
  }
});

function updateFromDivisions() {
  jumpsString.value = Array(divisions.value).fill("1").join(" ");
  degreesString.value = [...Array(divisions.value).keys()]
    .map((k) => (k + 1).toString())
    .join(" ");
}

function updateFromJumps() {
  if (jumps.value.includes(NaN)) {
    return;
  }
  const degrees_: string[] = [];
  let degree = 0;
  jumps.value.forEach((jump) => {
    degree += jump;
    degrees_.push(degree.toString());
  });
  divisions.value = degree;
  degreesString.value = degrees_.join(" ");
}

function updateFromDegrees() {
  if (degrees.value.includes(NaN)) {
    return;
  }
  const jumps_: string[] = [];
  let previous = 0;
  degrees.value.forEach((degree) => {
    jumps_.push((degree - previous).toString());
    previous = degree;
  });
  divisions.value = previous;
  jumpsString.value = jumps_.join(" ");
}

function generate() {
  const scale = Scale.fromEqualTemperamentSubset(degrees.value, equave.value);
  emit(
    "update:scaleName",
    `${divisions.value} equal divisions of ${equave.value.toString()}`
  );
  emit("update:scale", scale);
}
</script>

<template>
  <Modal @confirm="generate" @cancel="$emit('cancel')">
    <template #header>
      <h2>Generate equal temperament</h2>
    </template>
    <template #body>
      <div class="control-group">
        <div class="control">
          <label for="divisions">Number of divisions</label>
          <input
            id="divisions"
            type="number"
            min="1"
            v-model="divisions"
            @input="updateFromDivisions"
          />
        </div>
        <div class="control">
          <label for="jumps">Relative steps</label>
          <textarea
            ref="jumpsElement"
            id="jumps"
            v-model="jumpsString"
            @input="updateFromJumps"
          ></textarea>
        </div>
        <div class="control">
          <label for="degrees">Absolute degrees</label>
          <textarea
            ref="degreesElement"
            id="degrees"
            v-model="degreesString"
            @input="updateFromDegrees"
          ></textarea>
        </div>
        <div class="control">
          <label for="equave">Interval to divide</label>
          <ScaleLineInput
            id="equave"
            @update:value="equave = $event"
            v-model="equaveString"
          />
        </div>
      </div>
    </template>
  </Modal>
</template>