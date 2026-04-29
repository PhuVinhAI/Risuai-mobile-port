<script lang="ts">
    import { type character, type groupChat } from "src/ts/storage/database.svelte";
    import { DBState, MobileGUIStack } from "src/ts/stores.svelte";
    import BarIcon from "src/lib/SideBars/BarIcon.svelte";
    import { changeChar, getCharImage } from "src/ts/characters";
    import { addCharacter } from "src/ts/characters";
    import { language } from "src/lang";
    import { PlusIcon, MessageSquareIcon, CompassIcon, ChevronRightIcon } from "@lucide/svelte";

    const agoFormatter = new Intl.RelativeTimeFormat(navigator.languages, { style: "short" });

    function makeAgoText(time: number) {
        if (time === 0) return "";
        const diff = Date.now() - time;
        if (diff < 3600000) return agoFormatter.format(-Math.floor(diff / 60000), "minute");
        if (diff < 86400000) return agoFormatter.format(-Math.floor(diff / 3600000), "hour");
        if (diff < 604800000) return agoFormatter.format(-Math.floor(diff / 86400000), "day");
        if (diff < 2592000000) return agoFormatter.format(-Math.floor(diff / 604800000), "week");
        if (diff < 31536000000) return agoFormatter.format(-Math.floor(diff / 2592000000), "month");
        return agoFormatter.format(-Math.floor(diff / 31536000000), "year");
    }

    function getRecentCharacters(chars: (character | groupChat)[], limit: number) {
        return chars
            .map((c, i) => ({
                name: c.name || "Unnamed",
                image: c.image,
                chats: c.chats.length,
                i,
                interaction: c.lastInteraction || 0,
                agoText: makeAgoText(c.lastInteraction || 0),
            }))
            .filter((c) => c.interaction > 0)
            .sort((a, b) => b.interaction - a.interaction)
            .slice(0, limit);
    }

    let recentChars = $derived(getRecentCharacters(DBState.db.characters, 6));
    let totalChars = $derived(DBState.db.characters.length);
</script>

<div class="flex flex-col w-full h-full overflow-y-auto home-scroll">
    <!-- Greeting -->
    <div class="px-5 pt-6 pb-4">
        <h1 class="text-2xl font-bold text-textcolor">{language.home}, {DBState.db.username || "User"}</h1>
        <p class="text-sm text-textcolor2 mt-1">{totalChars} {language.character}{totalChars !== 1 ? "s" : ""}</p>
    </div>

    <!-- Quick Actions -->
    <div class="px-5 pb-4">
        <div class="flex gap-3">
            <button
                class="flex-1 flex items-center gap-3 p-4 rounded-2xl bg-darkbg transition-all duration-150 active:scale-[0.97] active:opacity-80"
                onclick={() => addCharacter()}
            >
                <div class="w-10 h-10 rounded-full bg-borderc/15 flex items-center justify-center">
                    <PlusIcon size={20} class="text-borderc" />
                </div>
                <div class="text-left">
                    <div class="text-sm font-semibold text-textcolor">{language.addCharacter}</div>
                    <div class="text-[11px] text-textcolor2">{language.character}</div>
                </div>
            </button>
            <button
                class="flex-1 flex items-center gap-3 p-4 rounded-2xl bg-darkbg transition-all duration-150 active:scale-[0.97] active:opacity-80"
                onclick={() => MobileGUIStack.set(2)}
            >
                <div class="w-10 h-10 rounded-full bg-borderc/15 flex items-center justify-center">
                    <CompassIcon size={20} class="text-borderc" />
                </div>
                <div class="text-left">
                    <div class="text-sm font-semibold text-textcolor">{language.hub}</div>
                    <div class="text-[11px] text-textcolor2">{language.search}</div>
                </div>
            </button>
        </div>
    </div>

    <!-- Recent Characters -->
    {#if recentChars.length > 0}
        <div class="px-5 pb-2">
            <div class="flex items-center justify-between mb-3">
                <h2 class="text-base font-semibold text-textcolor">{language.recent || "Recent"}</h2>
                <button
                    class="flex items-center gap-0.5 text-xs text-borderc font-medium transition-opacity active:opacity-60"
                    onclick={() => MobileGUIStack.set(1)}
                >
                    {language.recent}
                    <ChevronRightIcon size={14} />
                </button>
            </div>
            <div class="flex flex-col gap-1">
                {#each recentChars as char}
                    <button
                        class="flex items-center gap-3 p-3 rounded-2xl transition-all duration-150 active:bg-darkbg active:scale-[0.98]"
                        onclick={() => changeChar(char.i)}
                    >
                        <div class="w-12 h-12 rounded-full overflow-hidden shrink-0 ring-2 ring-darkborderc">
                            <BarIcon additionalStyle={getCharImage(char.image, "css")}></BarIcon>
                        </div>
                        <div class="flex-1 min-w-0 text-left">
                            <div class="text-sm font-semibold text-textcolor truncate">{char.name}</div>
                            <div class="text-xs text-textcolor2 flex items-center gap-1.5 mt-0.5">
                                <MessageSquareIcon size={11} />
                                <span>{char.chats}</span>
                                {#if char.agoText}
                                    <span class="text-textcolor2/40">·</span>
                                    <span>{char.agoText}</span>
                                {/if}
                            </div>
                        </div>
                        <ChevronRightIcon size={16} class="text-textcolor2/40 shrink-0" />
                    </button>
                {/each}
            </div>
        </div>
    {:else}
        <div class="flex-1 flex flex-col items-center justify-center px-8 pb-16">
            <div class="w-16 h-16 rounded-full bg-darkbg flex items-center justify-center mb-4">
                <MessageSquareIcon size={28} class="text-textcolor2/40" />
            </div>
            <p class="text-sm text-textcolor2 text-center">{language.noData}</p>
            <button
                class="mt-4 px-6 py-2.5 rounded-full bg-borderc text-white text-sm font-medium transition-all duration-150 active:scale-95 active:opacity-80"
                onclick={() => addCharacter()}
            >
                {language.addCharacter}
            </button>
        </div>
    {/if}
</div>

<style>
    .home-scroll {
        scrollbar-width: none;
        -ms-overflow-style: none;
    }
    .home-scroll::-webkit-scrollbar {
        display: none;
    }
</style>
