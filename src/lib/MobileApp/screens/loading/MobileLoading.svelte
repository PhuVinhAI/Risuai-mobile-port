<script lang="ts">
    import { LoadingStatusState } from "src/ts/stores.svelte";
</script>

<div class="fixed inset-0 w-full h-full bg-bgcolor flex flex-col items-center justify-center font-[system-ui]">
    <div class="flex flex-col items-center gap-6 loading-entrance">
        <div class="w-24 h-24">
            <img src="/logo_256.png" alt="RisuAI" class="w-24 h-24 object-cover" />
        </div>

        <div class="flex flex-col items-center gap-1.5">
            <h1 class="text-2xl font-bold text-textcolor tracking-tight">RisuAI</h1>
            <div class="flex gap-1">
                <span class="w-1.5 h-1.5 rounded-full bg-borderc dot dot-1"></span>
                <span class="w-1.5 h-1.5 rounded-full bg-borderc dot dot-2"></span>
                <span class="w-1.5 h-1.5 rounded-full bg-borderc dot dot-3"></span>
            </div>
        </div>
    </div>

    <div class="absolute bottom-12 left-0 right-0 text-center px-6">
        <span class="text-xs text-textcolor2">{LoadingStatusState.text}</span>
    </div>
</div>

<style>
    .loading-entrance {
        animation: entrance 0.6s cubic-bezier(0.16, 1, 0.3, 1);
    }
    @keyframes entrance {
        from { transform: scale(0.9) translateY(12px); opacity: 0; }
        to { transform: scale(1) translateY(0); opacity: 1; }
    }

    .spinning-ring {
        animation: spin 1.2s linear infinite;
    }
    @keyframes spin {
        to { transform: rotate(360deg); }
    }

    .dot {
        animation: dot-bounce 1.4s ease-in-out infinite;
    }
    .dot-2 { animation-delay: 0.16s; }
    .dot-3 { animation-delay: 0.32s; }
    @keyframes dot-bounce {
        0%, 80%, 100% { opacity: 0.3; transform: scale(0.8); }
        40% { opacity: 1; transform: scale(1.2); }
    }
</style>
