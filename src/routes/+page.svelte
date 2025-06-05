<script lang="ts">
    import { onMount } from "svelte";
    import { writable, derived } from "svelte/store";

    // Pilihan waktu preset dalam detik
    const presets = {
        pomodoro: 25 * 60,
        shortBreak: 5 * 60,
        longBreak: 15 * 60
    };

    // State utama
    const time = writable(presets.pomodoro); // waktu saat ini (detik)
    const isRunning = writable(false) // status timer aktif/tidak
    const currentMode = writable<'pomodoro' | 'shortBreak' | 'longBreak'>('pomodoro') //mode sekarang

    let interval: ReturnType<typeof setInterval>;
    let alarm: HTMLAudioElement;

    // Format detik menjadi string MM:SS
    const formattedTime = derived(time, $time => {
        const minutes = Math.floor($time / 60);
        const seconds = $time % 60;
        return `${String(minutes).padStart(2, '0')}:${String(seconds).padStart(2, '0')}`;
    });

    // Fungsi Start / Pause timer
    function startPause() {
        isRunning.update(running => {
            if (!running) {
                interval = setInterval(() => {
                    time.update(t => {
                        if (t > 0) return t - 1;
                        clearInterval(interval);
                        isRunning.set(false);
                        alarm.play(); // bunyikan alarm saat waktu habis
                        return 0;
                    });
                }, 1000);
            } else {
                clearInterval(interval);
            }
            return !running;
        });
    }

    // Reset ke preset sekarang
    function reset() {
        clearInterval(interval);
        isRunning.set(false);
        currentMode.update(mode => {
            time.set(presets[mode]);
            return mode;
        });
    }

    // Ubah mode (pomodoro / shortBreak / longBreak)
    function switchMode(mode: 'pomodoro' | 'shortBreak' | 'longBreak') {
        clearInterval(interval);
        isRunning.set(false);
        currentMode.set(mode);
        time.set(presets[mode]);
    }

    // Bersihkan interval saat component di-unmount
    onMount(() => {
        alarm = new Audio('/alarm.mp3'); // file alarm harus disimpan di static/
        return () => clearInterval(interval);
    });
</script>

<!-- Tampilan utama -->
<div class="min-h-screen flex flex-col items-center justify-center bg-red-50 text-red-800">
    <h1 class="text-4xl font-bold mb-6">Pomodoro Timer</h1>
    
    <!-- Pilihan mode -->
    <div class="flex gap-4 mb-6">
        <button class="px-4 py-2 rounded bg-red-500 text-white hover:bg-red-600 transition" on:click={() => switchMode('pomodoro')}>
            Pomodoro
        </button>
        <button class="px-4 py-2 rounded bg-blue-500 text-white hover:bg-blue-600 transition" on:click={() => switchMode('shortBreak')}>
            Short Break
        </button>
        <button class="px-4 py-2 rounded bg-green-500 text-white hover:bg-green-600 transition" on:click={() => switchMode('longBreak')}>
            Long Break
        </button>
    </div>

    <!-- Waktu -->
    <div class="text-7xl font-mono mb-6">
        {$formattedTime}
    </div>
    
    <!-- Tombol Kontrol -->
    <div class="flex gap-4">
        <button class="px-6 py-2 rounded-md bg-red-600 text-white hover:bg-red-700 transition" on:click={startPause}>
            {$isRunning ? 'Pause' : 'Start'}
        </button>

        <button class="px-6 py-2 rounded-md bg-gray-300 text-gray-800 hover:bg-gray-400 transition" on:click={reset}>
            Reset
        </button>
    </div>
</div>









