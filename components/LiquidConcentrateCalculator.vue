<script setup lang="ts">
const counter = useState('counter', () => Math.round(Math.random() * 1000))
const recommendedConcentration = ref(1) // tablespoons per gallon
const desiredStrength = ref(1) // %
const gallonsRequired = ref(1) // gallons

const desiredConcentration = computed(() => {
    return recommendedConcentration.value * desiredStrength.value
})
const amountSolidToAddInTbs = computed(() => {
    return desiredConcentration.value * gallonsRequired.value
})

const dialReading = 4.0 // oz / gal
const amountWaterToAddInOz = computed(() => {
    return dialReading / (desiredConcentration.value / amountSolidToAddInTbs.value)
})

const solidsVolumeToAddInTbs = computed(() => {
    return amountSolidToAddInTbs.value.toFixed(2)
})
const waterVolumeToAddInOz = computed(() => {
    return amountWaterToAddInOz.value.toFixed(2)
}) // max 32oz

const foo = computed(() => {
    return recommendedConcentration.value * desiredStrength.value * gallonsRequired.value
})
</script>

<template>
    <div class="h-screen m-0 px-16 bg-slate-300 grid grid-cols-1 gap-4">
        <div class="flex gap-4 place-content-center items-center justify-items-center">
            <div>Recommended tablespoons per gallon:</div>
            <div>
                <UInputNumber
                    v-model="recommendedConcentration"
                    :min="1"
                    :increment="{
                        color: 'neutral',
                        variant: 'solid',
                        size: 'xs'
                    }"
                    :decrement="{
                        color: 'neutral',
                        variant: 'solid',
                        size: 'xs'
                    }"
                    size="xl"
                />
            </div>
        </div>
        <div class="flex gap-4 place-content-center items-center justify-items-center">
            <div>Desired strength relative to recommended concentration:</div>
            <div>
                <UInputNumber
                    v-model="desiredStrength"
                    :min=".1"
                    :max="1"
                    :step="0.1"
                    :format-options="{
                    style: 'percent'
                    }"
                    :increment="{
                        color: 'neutral',
                        variant: 'solid',
                        size: 'xs'
                    }"
                    :decrement="{
                        color: 'neutral',
                        variant: 'solid',
                        size: 'xs'
                    }"
                    size="xl"
                />
            </div>
        </div>
        <div class="flex gap-4 place-content-center items-center justify-items-center">
            <div>Gallons of solution required:</div>
            <div>
                <UInputNumber
                    v-model="gallonsRequired"
                    :min="1"
                    :increment="{
                        color: 'neutral',
                        variant: 'solid',
                        size: 'xs'
                    }"
                    :decrement="{
                        color: 'neutral',
                        variant: 'solid',
                        size: 'xs'
                    }"
                    size="xl"
                />
            </div>
        </div>
        <div class="text-center">
            Add {{ solidsVolumeToAddInTbs }} tablespoons of product to {{ waterVolumeToAddInOz }} oz of water.
            Set dial to 4 oz / gal.
        </div>
    </div>
</template>
