<script setup lang="ts">
const counter = useState('counter', () => Math.round(Math.random() * 1000))
const recommendedConcentration = ref(1) // tablespoons per gallon
const desiredStrength = ref(1) // %
const gallonsRequired = ref(1) // gallons

// DIAL_READINGS is an ordered list of tuples where the first element is the concentration in oz per gallon and the second element is the English description.
const DIAL_READINGS: [number, string][] = [
    [8, "8 oz"],
    [6, "6 oz"],
    [16/3, "5 1/3 oz"],
    [4, "4 oz"],
    [3, "3 oz"],
    [5/2, "2.5 oz"],
    [2, "2 oz"],
    [3/2, "1.5 oz"],
    [1, "1 oz"],
    [2/3, "4 tsp"],
    [1/2, "1 tbs"],
    [1/4, "1.5 tsp"],
    [1/6, "1 tsp"],
]
const MAX_CONCENTRATE_CAPACITY = 32 // oz

interface Recipe {
    dialReadingToUse: string
    tbsSolidToAdd: string
    ozWaterToAdd: string
}

function calculateRecipe(): Recipe | undefined {
    // This must be the result of resulting liquid concentration (tbs / oz) multiplied by the dial setting (oz / gal)
    const desiredConcentration = recommendedConcentration.value * desiredStrength.value // tbs / gal

    // TODO: This can potentially be a fractional number, which is hard to measure. Maybe ignore desired gallons in such cases and make extra?
    const amountSolidToAddInTbs = desiredConcentration * gallonsRequired.value // tbs

    // We start with the maximum dial setting and work our way down to find the best dial setting
    for (const [dialReading, description] of DIAL_READINGS) {
        const amountWaterToAddInOz = dialReading / (desiredConcentration / amountSolidToAddInTbs) // oz

        // If the concentrate is > 50% solids by volume, it will be too thick to siphon into the sprayer.
        const isSludge = amountSolidToAddInTbs / amountWaterToAddInOz > 1

        if (amountWaterToAddInOz <= MAX_CONCENTRATE_CAPACITY && !isSludge) {
            return {
                dialReadingToUse: description,
                tbsSolidToAdd: amountSolidToAddInTbs.toFixed(2),
                ozWaterToAdd: amountWaterToAddInOz.toFixed(2),
            }
        }
    }
}

const recipe = computed(() => {
    return calculateRecipe()
})
</script>

<template>
    <div class="h-screen m-0 px-4 md:px-16 bg-slate-100 grid grid-cols-1 gap-4 text-slate-900">
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
            <p v-if="recipe">
            Add <b>{{ recipe.tbsSolidToAdd }} tablespoons</b> of product to <b>{{ recipe.ozWaterToAdd }} oz</b> of water.
            Set dial to <b>{{ recipe.dialReadingToUse }}</b>.
            </p>
            <p v-else>Your requested volume cannot be made with a single batch. Please try a lower volume or strength.</p>
        </div>
    </div>
</template>
