<script lang="ts">
    import { AccessibilityIcon, ActivityIcon, PackageIcon, BotIcon, BoxIcon, CodeIcon, ContactIcon, LanguagesIcon, MonitorIcon, Sailboat, UserIcon, ChevronRightIcon, SparkleIcon } from "@lucide/svelte";
    import { language } from "src/lang";
    import { additionalSettingsMenu, easyPanelStore, SettingsMenuIndex, DBState } from "src/ts/stores.svelte";
    import { isLite } from "src/ts/lite";
    import PluginDefinedIcon from "src/lib/Others/PluginDefinedIcon.svelte";

    import BotSettings from "src/lib/Setting/Pages/BotSettings.svelte";
    import PersonaSettings from "src/lib/Setting/Pages/PersonaSettings.svelte";
    import OtherBotSettings from "src/lib/Setting/Pages/OtherBotSettings.svelte";
    import DisplaySettings from "src/lib/Setting/Pages/DisplaySettings.svelte";
    import LanguageSettings from "src/lib/Setting/Pages/LanguageSettings.svelte";
    import AccessibilitySettings from "src/lib/Setting/Pages/AccessibilitySettings.svelte";
    import ModuleSettings from "src/lib/Setting/Pages/Module/ModuleSettings.svelte";
    import PluginSettings from "src/lib/Setting/Pages/PluginSettings.svelte";
    import UserSettings from "src/lib/Setting/Pages/UserSettings.svelte";
    import AdvancedSettings from "src/lib/Setting/Pages/AdvancedSettings.svelte";
    import ThanksPage from "src/lib/Setting/Pages/ThanksPage.svelte";
    import PromptSettings from "src/lib/Setting/Pages/PromptSettings.svelte";
    import FilesSettings from "src/lib/Setting/Pages/FilesSettings.svelte";
    import Communities from "src/lib/Setting/Pages/Communities.svelte";
    import GlobalLoreBookSettings from "src/lib/Setting/Pages/GlobalLoreBookSettings.svelte";
    import GlobalRegex from "src/lib/Setting/Pages/GlobalRegex.svelte";
    import Lorepreset from "src/lib/Setting/lorepreset.svelte";

    let openLoreList = $state(false);

    const groups = $derived([
        {
            title: "AI",
            items: [
                { idx: 1, icon: BotIcon, label: language.chatBot, color: "bg-blue-500" },
                { idx: 12, icon: ContactIcon, label: language.persona, color: "bg-violet-500" },
                { idx: 2, icon: Sailboat, label: language.otherBots, color: "bg-cyan-500" },
            ]
        },
        {
            title: language.display,
            items: [
                { idx: 3, icon: MonitorIcon, label: language.display, color: "bg-indigo-500" },
                { idx: 10, icon: LanguagesIcon, label: language.language, color: "bg-orange-500", lite: true },
                { idx: 11, icon: AccessibilityIcon, label: language.accessibility, color: "bg-teal-500" },
            ]
        },
        {
            title: language.plugin,
            items: [
                { idx: 14, icon: PackageIcon, label: language.modules, color: "bg-emerald-500" },
                { idx: 4, icon: CodeIcon, label: language.plugin, color: "bg-purple-500" },
            ]
        },
        {
            items: [
                { idx: 0, icon: UserIcon, label: `${language.account} & ${language.files}`, color: "bg-gray-500", lite: true },
                { idx: 6, icon: ActivityIcon, label: language.advancedSettings, color: "bg-red-500" },
                { idx: 77, icon: BoxIcon, label: language.supporterThanks, color: "bg-amber-500" },
            ]
        },
    ]);

    function getVisibleItems(items: typeof groups[0]['items']) {
        return items.filter(item => !$isLite || item.lite);
    }
</script>

{#if $SettingsMenuIndex === -1}
    <div class="flex flex-col w-full px-4 pt-2 pb-6 gap-6">
        {#each groups as group}
            {@const visible = getVisibleItems(group.items)}
            {#if visible.length > 0}
                {#if group.title}
                    <div class="text-xs font-semibold text-textcolor2 uppercase tracking-wider px-1 mb-[-16px]">{group.title}</div>
                {/if}
                <div class="rounded-2xl bg-darkbg overflow-hidden">
                    {#each visible as item, i}
                        <button
                            class="w-full flex items-center gap-3 px-4 py-3.5 text-left transition-all active:scale-[0.98] active:opacity-70"
                            class:border-t={i > 0}
                            class:border-darkborderc={i > 0}
                            onclick={() => $SettingsMenuIndex = item.idx}
                        >
                            <div class="w-8 h-8 rounded-lg {item.color} flex items-center justify-center shrink-0">
                                <item.icon size={18} color="white" strokeWidth={2} />
                            </div>
                            <span class="flex-1 text-sm text-textcolor font-medium">{item.label}</span>
                            <ChevronRightIcon size={16} class="text-textcolor2 opacity-40" />
                        </button>
                    {/each}
                </div>
            {/if}
        {/each}

        {#if additionalSettingsMenu.length > 0}
            <div class="rounded-2xl bg-darkbg overflow-hidden">
                {#each additionalSettingsMenu as menu, i}
                    <button
                        class="w-full flex items-center gap-3 px-4 py-3.5 text-left transition-all active:scale-[0.98] active:opacity-70"
                        class:border-t={i > 0}
                        class:border-darkborderc={i > 0}
                        onclick={() => menu.callback()}
                    >
                        <div class="w-8 h-8 rounded-lg bg-slate-500 flex items-center justify-center shrink-0">
                            <PluginDefinedIcon ico={menu} />
                        </div>
                        <span class="flex-1 text-sm text-textcolor font-medium">{menu.name}</span>
                        <ChevronRightIcon size={16} class="text-textcolor2 opacity-40" />
                    </button>
                {/each}
            </div>
        {/if}

        {#if DBState.db.enableRisuaiProTools && !$isLite}
            <div class="rounded-2xl bg-darkbg overflow-hidden">
                <button
                    class="w-full flex items-center gap-3 px-4 py-3.5 text-left transition-all active:scale-[0.98] active:opacity-70"
                    onclick={() => easyPanelStore.open = true}
                >
                    <div class="w-8 h-8 rounded-lg bg-gradient-to-r from-blue-500 to-teal-500 flex items-center justify-center shrink-0">
                        <SparkleIcon size={18} color="white" strokeWidth={2} />
                    </div>
                    <span class="flex-1 text-sm text-textcolor font-medium">{language.easyPanel}</span>
                    <ChevronRightIcon size={16} class="text-textcolor2 opacity-40" />
                </button>
            </div>
        {/if}

        <div class="text-center text-[11px] text-textcolor2 opacity-40 py-2">RisuAI</div>
    </div>

{:else}
    {#key $SettingsMenuIndex}
        <div class="flex flex-col w-full px-4 py-4 text-textcolor min-w-0">
            {#if $SettingsMenuIndex === 0}
                <UserSettings />
            {:else if $SettingsMenuIndex === 1}
                <BotSettings goPromptTemplate={() => $SettingsMenuIndex = 13} />
            {:else if $SettingsMenuIndex === 2}
                <OtherBotSettings />
            {:else if $SettingsMenuIndex === 3}
                <DisplaySettings />
            {:else if $SettingsMenuIndex === 4}
                <PluginSettings />
            {:else if $SettingsMenuIndex === 5}
                <FilesSettings />
            {:else if $SettingsMenuIndex === 6}
                <AdvancedSettings />
            {:else if $SettingsMenuIndex === 7}
                <Communities />
            {:else if $SettingsMenuIndex === 8}
                <GlobalLoreBookSettings bind:openLoreList />
            {:else if $SettingsMenuIndex === 9}
                <GlobalRegex />
            {:else if $SettingsMenuIndex === 10}
                <LanguageSettings />
            {:else if $SettingsMenuIndex === 11}
                <AccessibilitySettings />
            {:else if $SettingsMenuIndex === 12}
                <PersonaSettings />
            {:else if $SettingsMenuIndex === 13}
                <PromptSettings onGoBack={() => $SettingsMenuIndex = 1} />
            {:else if $SettingsMenuIndex === 14}
                <ModuleSettings />
            {:else if $SettingsMenuIndex === 77}
                <ThanksPage />
            {/if}
        </div>
    {/key}
{/if}

{#if openLoreList}
    <Lorepreset close={() => openLoreList = false} />
{/if}
