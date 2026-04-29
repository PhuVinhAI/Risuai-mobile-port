<script lang="ts">
    import { ArrowLeft, MenuIcon, SearchIcon, XIcon } from "@lucide/svelte";
    import { language } from "src/lang";
    import { DBState, MobileGUIStack, MobileSearch, MobileSideBar, selectedCharID, SettingsMenuIndex, CharConfigSubMenu } from "src/ts/stores.svelte";
    import { isLite } from "src/ts/lite";

    import BottomNavBar from "./components/BottomNavBar.svelte";
    import MobileHome from "./screens/home/MobileHome.svelte";
    import MobileCharacters from "src/lib/Mobile/MobileCharacters.svelte";
    import RealmMain from "src/lib/UI/Realm/RealmMain.svelte";
    import Settings from "src/lib/Setting/Settings.svelte";
    import MobileSettings from "./screens/settings/MobileSettings.svelte";
    import ChatScreen from "src/lib/ChatScreens/ChatScreen.svelte";
    import SideChatList from "src/lib/SideBars/SideChatList.svelte";
    import CharConfig from "src/lib/SideBars/CharConfig.svelte";
    import DevTool from "src/lib/SideBars/DevTool.svelte";

    import { WrenchIcon, UserIcon, SmileIcon, BookIcon, Volume2Icon, Braces, ActivityIcon } from "@lucide/svelte";

    let searchOpen = $state(false);
</script>

<div class="w-full h-full flex flex-col bg-bgcolor font-[system-ui]">

    <!-- ===== HEADER ===== -->
    <header class="shrink-0 w-full bg-darkbg border-b border-darkborderc z-20" style="padding-top: env(safe-area-inset-top, 0px);">
        <div class="flex items-center h-14 px-4 gap-3">

            {#if $selectedCharID !== -1 && $MobileSideBar > 0}
                <!-- Sidebar mode header -->
                <button class="shrink-0 w-9 h-9 rounded-full flex items-center justify-center active:scale-90 active:opacity-60 transition-all" onclick={() => MobileSideBar.set(0)}>
                    <ArrowLeft size={22} />
                </button>
                <span class="font-bold text-[17px] truncate">{language.menu}</span>

            {:else if $selectedCharID !== -1}
                <!-- Chat mode header -->
                <button class="shrink-0 w-9 h-9 rounded-full flex items-center justify-center active:scale-90 active:opacity-60 transition-all" onclick={() => selectedCharID.set(-1)}>
                    <ArrowLeft size={22} />
                </button>
                <span class="font-bold text-[17px] flex-1 min-w-0 truncate">{DBState.db.characters[$selectedCharID]?.name || ""}</span>
                <button class="shrink-0 w-9 h-9 rounded-full flex items-center justify-center active:scale-90 active:opacity-60 transition-all" onclick={() => MobileSideBar.set(1)}>
                    <MenuIcon size={22} />
                </button>

            {:else if $MobileGUIStack === 3 && $SettingsMenuIndex > -1}
                <!-- Settings sub-menu header -->
                <button class="shrink-0 w-9 h-9 rounded-full flex items-center justify-center active:scale-90 active:opacity-60 transition-all" onclick={() => SettingsMenuIndex.set(-1)}>
                    <ArrowLeft size={22} />
                </button>
                <span class="font-bold text-[17px]">{language.settings}</span>

            {:else if $MobileGUIStack === 1}
                <!-- Characters tab header with search -->
                {#if searchOpen}
                    <div class="flex-1 flex items-center gap-2">
                        <input
                            placeholder="{language.search}..."
                            bind:value={$MobileSearch}
                            class="flex-1 bg-transparent text-textcolor text-base outline-none placeholder:text-textcolor2/40"
                            autofocus
                        />
                        <button class="shrink-0 w-9 h-9 rounded-full flex items-center justify-center active:scale-90 active:opacity-60 transition-all" onclick={() => { searchOpen = false; $MobileSearch = ""; }}>
                            <XIcon size={20} />
                        </button>
                    </div>
                {:else}
                    <span class="font-bold text-[17px] flex-1">{language.character}</span>
                    <button class="shrink-0 w-9 h-9 rounded-full flex items-center justify-center active:scale-90 active:opacity-60 transition-all" onclick={() => searchOpen = true}>
                        <SearchIcon size={20} />
                    </button>
                {/if}

            {:else if $MobileGUIStack === 2}
                <!-- Browse tab header -->
                <span class="font-bold text-[17px]">{language.hub}</span>

            {:else if $MobileGUIStack === 3}
                <!-- Settings tab header -->
                <span class="font-bold text-[17px]">{language.settings}</span>

            {:else}
                <!-- Home tab header -->
                <img src="/logo_256.png" alt="RisuAI" class="w-8 h-8 rounded-lg" />
                <span class="font-bold text-[17px]">RisuAI</span>
            {/if}
        </div>
    </header>

    <!-- ===== SIDEBAR TAB BAR (when in chat sidebar) ===== -->
    {#if $selectedCharID !== -1 && $MobileSideBar > 0 && !$isLite}
        <div class="shrink-0 w-full px-2 py-1.5 text-textcolor2 border-b border-darkborderc bg-darkbg flex justify-start items-center gap-1">
            <button class="flex-1 py-1.5 rounded-lg text-sm font-medium transition-colors" class:text-textcolor={$MobileSideBar === 1} class:bg-bgcolor={$MobileSideBar === 1} onclick={() => $MobileSideBar = 1}>
                {language.Chat}
            </button>
            <button class="flex-1 py-1.5 rounded-lg text-sm font-medium transition-colors" class:text-textcolor={$MobileSideBar === 2} class:bg-bgcolor={$MobileSideBar === 2} onclick={() => $MobileSideBar = 2}>
                {language.character}
            </button>
            <button class="w-10 py-1.5 rounded-lg flex items-center justify-center transition-colors" class:text-textcolor={$MobileSideBar === 3} class:bg-bgcolor={$MobileSideBar === 3} onclick={() => $MobileSideBar = 3}>
                <WrenchIcon size={16} />
            </button>
        </div>
    {/if}

    <!-- ===== BODY ===== -->
    <div class="flex-1 w-full overflow-y-auto bg-bgcolor relative">
        {#if $selectedCharID !== -1 && $MobileSideBar > 0}
            <!-- Sidebar panels -->
            <div class="w-full flex flex-col p-2 mt-2 h-full">
                {#if $MobileSideBar === 1}
                    <SideChatList bind:chara={DBState.db.characters[$selectedCharID]} />
                {:else if $MobileSideBar === 2}
                    <CharConfig />
                {:else if $MobileSideBar === 3}
                    <DevTool />
                {/if}
            </div>
        {:else if $selectedCharID !== -1}
            <!-- Chat screen -->
            <ChatScreen />
        {:else if $MobileGUIStack === 0}
            <MobileHome />
        {:else if $MobileGUIStack === 1}
            <MobileCharacters />
        {:else if $MobileGUIStack === 2}
            <RealmMain />
        {:else if $MobileGUIStack === 3}
            <MobileSettings />
        {/if}
    </div>

    <!-- ===== BOTTOM NAV / CHAR CONFIG TABS ===== -->
    {#if $selectedCharID === -1}
        <BottomNavBar />
    {/if}

    {#if $selectedCharID !== -1 && $MobileSideBar === 2}
        <div class="shrink-0 w-full px-2 py-3 border-t border-darkborderc bg-darkbg flex items-center justify-center text-textcolor2" style="padding-bottom: calc(env(safe-area-inset-bottom, 0px) + 12px);">
            <button class="flex justify-center items-center flex-col gap-1 w-14 max-w-14 transition-colors" class:text-textcolor={$CharConfigSubMenu === 0} onclick={() => CharConfigSubMenu.set(0)}>
                <UserIcon size={20} />
                <span class="text-[9px] truncate max-w-14">{language.basicInfo}</span>
            </button>
            <button class="flex justify-center items-center flex-col gap-1 w-14 max-w-14 transition-colors" class:text-textcolor={$CharConfigSubMenu === 1} onclick={() => CharConfigSubMenu.set(1)}>
                <SmileIcon size={20} />
                <span class="text-[9px] truncate max-w-14">{language.characterDisplay}</span>
            </button>
            <button class="flex justify-center items-center flex-col gap-1 w-14 max-w-14 transition-colors" class:text-textcolor={$CharConfigSubMenu === 3} onclick={() => CharConfigSubMenu.set(3)}>
                <BookIcon size={20} />
                <span class="text-[9px] truncate max-w-14">{language.loreBook}</span>
            </button>
            <button class="flex justify-center items-center flex-col gap-1 w-14 max-w-14 transition-colors" class:text-textcolor={$CharConfigSubMenu === 5} onclick={() => CharConfigSubMenu.set(5)}>
                <Volume2Icon size={20} />
                <span class="text-[9px] truncate max-w-14">TTS</span>
            </button>
            <button class="flex justify-center items-center flex-col gap-1 w-14 max-w-14 transition-colors" class:text-textcolor={$CharConfigSubMenu === 4} onclick={() => CharConfigSubMenu.set(4)}>
                <Braces size={20} />
                <span class="text-[9px] truncate max-w-14">{language.scripts}</span>
            </button>
            <button class="flex justify-center items-center flex-col gap-1 w-14 max-w-14 transition-colors" class:text-textcolor={$CharConfigSubMenu === 2} onclick={() => CharConfigSubMenu.set(2)}>
                <ActivityIcon size={20} />
                <span class="text-[9px] truncate max-w-14">{language.advanced}</span>
            </button>
        </div>
    {/if}
</div>
