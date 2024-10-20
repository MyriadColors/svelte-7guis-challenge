<script lang="ts">
    import { ModeWatcher } from "mode-watcher";
    import { Button } from "$lib/components/ui/button/index.js";
    import { Input } from "$lib/components/ui/input/index.js";
    import { Progress } from "$lib/components/ui/progress";
    import Slider from "$lib/components/ui/slider/slider.svelte";
    import Card from "$lib/components/ui/card/card.svelte";
    import Label from "$lib/components/ui/label/label.svelte";

    // Counter State
    let counter = $state(0);
    
    // Temperature State
    let roundFloat = (num: number, precision: number) => {
        const factor = Math.pow(10, precision);
        return Math.round(num * factor) / factor;
    }
    class Temperature {
        #c: number = $state(0);
        #f: number = $state(0);

        get celsius() {
            return this.#c;
        }

        set celsius(newValue) {
            this.#c = newValue;
            this.#f = roundFloat(newValue * 9 / 5 + 32, 2);
        }

        get fahrenheit() {
            return this.#f;
        }

        set fahrenheit(newValue) {
            this.#f = newValue;
            this.#c = roundFloat((newValue - 32) * 5 / 9, 2);
        }
    }

    const temperature = new Temperature();

    // Timer State
    let elapsed: number = $state(0);
    let duration: number = $state(5);
    let interval: number;

    function start() {
        interval = setInterval(() => {
            elapsed += 0.1;
            if (elapsed >= duration) {
                clearInterval(interval);
            }
        }, 100);
    }

    function reset() {
        elapsed = 0;  // Reset elapsed to 0 instead of duration
        $effect(() => {
            if (!duration) return;
            start();
            return () => clearInterval(interval);
        });
    }

    // Tailwind Classes
    const mainContainerClass = "flex min-h-screen flex-col items-center justify-center bg-slate-900";
    const formClass = "flex w-full max-w-md p-6 m-4 flex-col items-center justify-center space-y-6 bg-slate-800 rounded-lg shadow-lg border border-slate-700";
    const inputClass = "w-full text-center text-lg font-semibold bg-slate-700 text-white border-none rounded-md focus:ring-2 focus:ring-blue-500";
    const buttonClass = "w-full h-12 text-lg font-semibold bg-blue-600 text-white rounded-md hover:bg-blue-500 focus:ring-4 focus:ring-blue-400 transition-all";
    const labelClass = "text-white font-medium text-lg w-1/3";
    
    // Function to handle slider change
    function updateDuration(value: number) {
        duration = value;
    }
</script>

<ModeWatcher />
<main class={mainContainerClass}>
    <!-- Counter Form -->
    <form class={formClass}>
        <Input class={inputClass} disabled type="number" bind:value={counter} />
        <Button class={buttonClass} onclick={() => counter++}>Increase</Button>
    </form>

    <!-- Temperature Converter Form -->
    <form class={formClass}>
        <div class="flex w-full space-x-4">
            <div class="flex flex-col w-1/2 space-y-2">
                <label class={labelClass} for="celsius">Celsius</label>
                <Input class={inputClass} id="celsius" type="number" bind:value={temperature.celsius} />
            </div>
            <div class="flex flex-col w-1/2 space-y-2">
                <label class={labelClass} for="fahrenheit">Fahrenheit</label>
                <Input class={inputClass} id="fahrenheit" type="number" bind:value={temperature.fahrenheit} />
            </div>
        </div>
    </form>

    <!-- Timer Card -->
    <Card class="w-full max-w-md p-6 m-4 flex-col items-center justify-center space-y-6 bg-slate-800 rounded-lg shadow-lg border border-slate-700">
        <div class="flex flex-col w-full space-y-4">
            <Label class={labelClass} for="duration">Duration: {duration}s</Label>
            <!-- Couldn't make this work with the shadCn Slider component-->
            <input type="range" bind:value={duration} min="1" max="10" />
            <Label class={labelClass} for="progress">Elapsed: {roundFloat(elapsed, 2)}s</Label>
            <Progress id="progress" class="w-full" value={elapsed} max={duration} />
            <div class="flex w-full space-x-2">
                <Button onclick={start} class={buttonClass}>Start</Button>
                <Button onclick={reset} class={buttonClass}>Reset</Button>
            </div>
        </div>
    </Card>
</main>
