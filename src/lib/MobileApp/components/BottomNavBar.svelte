<script lang="ts">
    import { HomeIcon, UsersIcon, CompassIcon, SettingsIcon } from "@lucide/svelte";
    import { language } from "src/lang";
    import { MobileGUIStack } from "src/ts/stores.svelte";

    const tabs = $derived([
        { id: 0, icon: HomeIcon, label: language.home },
        { id: 1, icon: UsersIcon, label: language.character },
        { id: 2, icon: CompassIcon, label: language.hub },
        { id: 3, icon: SettingsIcon, label: language.settings },
    ]);
</script>

<nav class="shrink-0 w-full bg-darkbg border-t border-darkborderc backdrop-blur-xl" style="padding-bottom: env(safe-area-inset-bottom, 0px);">
    <div class="relative flex items-stretch justify-around h-16">
        <!-- Sliding pill -->
        <div
            class="absolute top-0 h-[3px] w-1/4 transition-all duration-300 ease-[cubic-bezier(0.34,1.56,0.64,1)]"
            style="left: {$MobileGUIStack * 25}%"
        >
            <div class="mx-auto w-8 h-full rounded-b bg-borderc"></div>
        </div>

        {#each tabs as tab}
            {@const active = $MobileGUIStack === tab.id}
            <button
                class="flex-1 flex flex-col items-center justify-center gap-0.5 transition-colors duration-200 active:scale-90"
                class:text-borderc={active}
                class:text-textcolor2={!active}
                class:opacity-40={!active}
                onclick={() => MobileGUIStack.set(tab.id)}
            >
                <tab.icon size={22} strokeWidth={active ? 2.5 : 1.8} />
                <span class="text-[10px]" class:font-bold={active} class:font-medium={!active}>{tab.label}</span>
            </button>
        {/each}
    </div>
</nav>
