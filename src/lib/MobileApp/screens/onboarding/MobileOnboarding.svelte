<script lang="ts">
    import { changeLanguage, language } from "src/lang";
    import { setPreset } from "src/ts/storage/database.svelte";
    import { DBState } from "src/ts/stores.svelte";
    import { prebuiltPresets } from "src/ts/process/templates/templates";
    import { updateTextThemeAndCSS } from "src/ts/gui/colorscheme";
    import { alertError } from "src/ts/alert";
    import { Send, ChevronLeft } from "@lucide/svelte";
    import Airisu from "../../../../etc/Airisu.webp";
    import { tick } from "svelte";

    let step = $state(0);
    let input = $state("");
    let provider = $state("");
    let chatLang = $state(0);
    let chatMemorySelection = $state(0);
    let chatContainer: HTMLDivElement;
    let stepHistory: number[] = $state([]);

    function goTo(nextStep: number) {
        stepHistory.push(step);
        step = nextStep;
    }

    function goBack() {
        if (stepHistory.length > 0) {
            step = stepHistory.pop()!;
            input = "";
            scrollToBottom();
        }
    }

    {
        const browserLang = navigator.language;
        const browserLangShort = browserLang.split("-")[0];
        const usableLangs = ["de", "en", "ko", "cn", "vi", "zh-Hant"];
        if (usableLangs.includes(browserLangShort)) {
            changeLanguage(browserLangShort);
            DBState.db.language = browserLangShort;
            goTo(1);
        }
    }

    async function scrollToBottom() {
        await tick();
        if (chatContainer) {
            chatContainer.scrollTo({ top: chatContainer.scrollHeight, behavior: "smooth" });
        }
    }

    function send() {
        switch (step) {
            case 1: {
                if (input.length > 0) {
                    DBState.db.username = input;
                    goTo(2);
                    input = "";
                }
                break;
            }
            case 4: {
                if (input.length === 0) break;
                if (!input.startsWith("sk-")) {
                    alertError("Invalid API key");
                    break;
                }
                if (provider === "openai") DBState.db.openAIKey = input;
                if (provider === "openrouter") DBState.db.openrouterKey = input;
                if (provider === "claude") DBState.db.claudeAPIKey = input;
                goTo(5);
                input = "";
                break;
            }
        }
        scrollToBottom();
    }

    function selectProvider(p: string) {
        provider = p;
        if (p === "horde") {
            goTo(5);
        } else if (p === "later") {
            goTo(10);
        } else {
            goTo(4);
        }
        scrollToBottom();
    }

    function selectChatLang(choice: number) {
        chatLang = choice;
        goTo(6);
        scrollToBottom();
    }

    function selectMemory(choice: number) {
        chatMemorySelection = choice;
        goTo(10);
        scrollToBottom();
    }

    $effect.pre(() => {
        if (step === 10) {
            setTimeout(() => {
                DBState.db = setPreset(DBState.db, prebuiltPresets.OAI2);
                DBState.db.textTheme = "highcontrast";
                updateTextThemeAndCSS();

                const memConfigs = [
                    { maxContext: 16000, maxResponse: 1000 },
                    { maxContext: 8000, maxResponse: 500 },
                    { maxContext: 12000, maxResponse: 800 },
                    { maxContext: 100000, maxResponse: 1000 },
                ];
                const mem = memConfigs[chatMemorySelection] ?? memConfigs[2];
                DBState.db.maxContext = mem.maxContext;
                DBState.db.maxResponse = mem.maxResponse;

                const modelMap: Record<string, { ai: string; sub: string }> = {
                    claude: { ai: "claude-3-5-sonnet-20241022", sub: "claude-3-5-sonnet-20241022" },
                    openai: { ai: "gpt4o-chatgpt", sub: "gpt4o-chatgpt" },
                    openrouter: { ai: "openrouter", sub: "openrouter" },
                    horde: { ai: "horde:::auto", sub: "horde:::auto" },
                };
                if (modelMap[provider]) {
                    DBState.db.aiModel = modelMap[provider].ai;
                    DBState.db.subModel = modelMap[provider].sub;
                }
                if (provider === "openrouter") DBState.db.openrouterRequestModel = "risu/free";

                if (chatLang !== 0) {
                    const langMap: Record<string, string> = { de: "de", en: "en", ko: "ko", cn: "zh", vi: "vi", "zh-Hant": "zh-TW" };
                    if (langMap[DBState.db.language]) DBState.db.translator = langMap[DBState.db.language];
                }
                if (chatLang === 1) {
                    DBState.db.autoTranslate = true;
                    DBState.db.translatorType = "google";
                    DBState.db.useAutoTranslateInput = true;
                }

                DBState.db.claudeCachingExperimental = true;
                DBState.db.didFirstSetup = true;
            }, 1500);
        }
    });

    $effect(() => {
        step;
        scrollToBottom();
    });

    const languages = [
        { code: "en", label: "English" },
        { code: "ko", label: "한국어" },
        { code: "cn", label: "中文" },
        { code: "zh-Hant", label: "繁體中文" },
        { code: "de", label: "Deutsch" },
        { code: "vi", label: "Tiếng Việt" },
    ];
</script>

<div class="fixed inset-0 w-full h-full bg-bgcolor text-textcolor flex flex-col font-[system-ui]">
    <!-- Header -->
    <div class="shrink-0 flex items-center gap-3 px-4 py-2.5 bg-darkbg z-10" style="padding-top: calc(env(safe-area-inset-top, 8px) + 8px);">
        {#if stepHistory.length > 0 && step !== 10}
            <button class="shrink-0 w-9 h-9 rounded-full flex items-center justify-center transition-all duration-200 active:scale-85 active:opacity-50" onclick={goBack}>
                <ChevronLeft size={22} />
            </button>
        {/if}
        <div class="w-10 h-10 rounded-full overflow-hidden shrink-0">
            <img src={Airisu} alt="Airisu" class="w-full h-full object-cover" />
        </div>
        <div class="flex-1 min-w-0">
            <div class="text-[15px] font-bold truncate">Airisu</div>
            <div class="text-[11px] text-textcolor2/50">RisuAI Assistant</div>
        </div>
    </div>

    <!-- Chat area -->
    <div class="flex-1 overflow-y-auto px-4 py-4 flex flex-col gap-3 ob-chat-area" bind:this={chatContainer}>

        <!-- Step 0: Language selection -->
        {#if step === 0}
            <div class="max-w-[85%] px-[18px] py-3.5 rounded-[20px] rounded-bl-[6px] bg-darkbg text-sm leading-relaxed self-start animate-ob-msg">
                <span class="font-medium">Choose your language</span>
            </div>
            <div class="flex flex-wrap gap-2 animate-ob-msg-delay">
                {#each languages as lang}
                    <button
                        class="px-4 py-2.5 rounded-2xl bg-darkbg text-sm font-medium text-textcolor transition-all duration-150 active:scale-95 active:opacity-80"
                        onclick={() => { changeLanguage(lang.code); DBState.db.language = lang.code; goTo(1); }}
                    >
                        {lang.label}
                    </button>
                {/each}
            </div>
        {/if}

        <!-- Step 1+: Welcome -->
        {#if step >= 1}
            <div class="max-w-[85%] px-[18px] py-3.5 rounded-[20px] rounded-bl-[6px] bg-darkbg text-sm leading-relaxed self-start animate-ob-msg">
                {language.setup.welcome}
            </div>
        {/if}

        <!-- Step 1: Username input (handled by bottom input) -->

        <!-- Step 2+: Username echo + setup choice -->
        {#if step >= 2}
            <div class="max-w-[75%] px-[18px] py-3.5 rounded-[20px] rounded-br-[6px] bg-borderc text-white text-sm leading-relaxed self-end animate-ob-msg">
                {DBState.db.username}
            </div>
            <div class="max-w-[85%] px-[18px] py-3.5 rounded-[20px] rounded-bl-[6px] bg-darkbg text-sm leading-relaxed self-start animate-ob-msg">
                {language.setup.setupLaterMessage.replace("{username}", DBState.db.username)}
            </div>
        {/if}

        {#if step === 2}
            <div class="flex flex-col gap-2 animate-ob-msg-delay">
                <button
                    class="w-full px-[18px] py-4 rounded-2xl bg-darkbg text-left text-textcolor cursor-pointer transition-all duration-200 active:scale-[0.97] active:opacity-80"
                    onclick={() => { goTo(3); scrollToBottom(); }}
                >
                    <div class="font-semibold text-[15px]">{language.setup.setupMessageOption1}</div>
                    <div class="text-xs text-textcolor2 mt-0.5">{language.setup.setupMessageOption1Desc}</div>
                </button>
                <button
                    class="w-full px-[18px] py-4 rounded-2xl bg-darkbg text-left text-textcolor cursor-pointer transition-all duration-200 active:scale-[0.97] active:opacity-80"
                    onclick={() => { provider = "later"; goTo(10); }}
                >
                    <div class="font-medium text-sm text-textcolor2">{language.setup.setupMessageOption2}</div>
                </button>
            </div>
        {/if}

        <!-- Step 3+: Provider selection -->
        {#if step === 3 || (step > 3 && stepHistory.includes(3))}
            <div class="max-w-[75%] px-[18px] py-3.5 rounded-[20px] rounded-br-[6px] bg-borderc text-white text-sm leading-relaxed self-end animate-ob-msg">
                {language.setup.setupMessageOption1}
            </div>
            <div class="max-w-[85%] px-[18px] py-3.5 rounded-[20px] rounded-bl-[6px] bg-darkbg text-sm leading-relaxed self-start animate-ob-msg">
                {language.setup.welcome2.replace("{username}", DBState.db.username)}
            </div>
        {/if}

        {#if step === 3}
            <div class="flex flex-col gap-2 animate-ob-msg-delay">
                <button class="w-full px-[18px] py-4 rounded-2xl bg-darkbg text-left text-textcolor cursor-pointer transition-all duration-200 active:scale-[0.97] active:opacity-80" onclick={() => selectProvider("claude")}>
                    <div class="flex items-center gap-2 font-semibold text-[15px]">Claude <span class="text-[10px] font-medium px-2 py-0.5 rounded-full bg-borderc/15 text-borderc">{language.recommended}</span></div>
                    <div class="text-xs text-textcolor2 mt-0.5">{language.setup.claudeDesc}</div>
                </button>
                <button class="w-full px-[18px] py-4 rounded-2xl bg-darkbg text-left text-textcolor cursor-pointer transition-all duration-200 active:scale-[0.97] active:opacity-80" onclick={() => selectProvider("openai")}>
                    <div class="font-semibold text-[15px]">OpenAI</div>
                    <div class="text-xs text-textcolor2 mt-0.5">{language.setup.openAIDesc}</div>
                </button>
                <button class="w-full px-[18px] py-4 rounded-2xl bg-darkbg text-left text-textcolor cursor-pointer transition-all duration-200 active:scale-[0.97] active:opacity-80" onclick={() => selectProvider("horde")}>
                    <div class="font-semibold text-[15px]">Horde</div>
                    <div class="text-xs text-textcolor2 mt-0.5">{language.setup.hordeProvider}</div>
                </button>
                <button class="w-full px-[18px] py-4 rounded-2xl bg-darkbg text-left text-textcolor cursor-pointer transition-all duration-200 active:scale-[0.97] active:opacity-80" onclick={() => selectProvider("openrouter")}>
                    <div class="font-semibold text-[15px]">OpenRouter</div>
                    <div class="text-xs text-textcolor2 mt-0.5">{language.setup.openRouterProvider}</div>
                </button>
            </div>
        {/if}

        <!-- Step 4+: API Key -->
        {#if step === 4 || (step > 4 && stepHistory.includes(4))}
            <div class="max-w-[75%] px-[18px] py-3.5 rounded-[20px] rounded-br-[6px] bg-borderc text-white text-sm leading-relaxed self-end animate-ob-msg">{provider}</div>
            <div class="max-w-[85%] px-[18px] py-3.5 rounded-[20px] rounded-bl-[6px] bg-darkbg text-sm leading-relaxed self-start animate-ob-msg whitespace-pre-line">
                {#if provider === "openai"}
                    {language.setup.setupOpenAI}
                {:else if provider === "openrouter"}
                    {language.setup.setupOpenRouter}
                {:else if provider === "claude"}
                    {language.setup.setupClaude}
                    {#each language.setup.setupClaudeSteps as cstep, i}
                        <div class="mt-2 text-sm">{i + 1}. {cstep}</div>
                    {/each}
                {/if}
            </div>
        {/if}

        {#if step === 4}
            <div class="text-xs text-textcolor2 text-center animate-ob-msg-delay">
                {language.setup.inputApiKey}
            </div>
            <button
                class="self-center mt-1 px-4 py-2 rounded-2xl text-sm text-textcolor2 transition-all duration-150 active:scale-95 active:bg-selected animate-ob-msg-delay"
                onclick={() => { goTo(5); scrollToBottom(); }}
            >
                {language.remindIgnore || "Skip"}
            </button>
        {/if}

        <!-- Step 5+: Chat language -->
        {#if step === 5 || (step > 5 && stepHistory.includes(5))}
            <div class="max-w-[85%] px-[18px] py-3.5 rounded-[20px] rounded-bl-[6px] bg-darkbg text-sm leading-relaxed self-start animate-ob-msg">
                {language.setup.chooseChatType}
            </div>
        {/if}

        {#if step === 5}
            <div class="flex flex-col gap-2 animate-ob-msg-delay">
                <button class="w-full px-[18px] py-4 rounded-2xl bg-darkbg text-left text-textcolor cursor-pointer transition-all duration-200 active:scale-[0.97] active:opacity-80" onclick={() => selectChatLang(0)}>
                    <div class="font-semibold text-[15px]">{language.setup.chooseChatTypeOption1}</div>
                    <div class="text-xs text-textcolor2 mt-0.5">{language.setup.chooseChatTypeOption1Desc}</div>
                </button>
                <button class="w-full px-[18px] py-4 rounded-2xl bg-darkbg text-left text-textcolor cursor-pointer transition-all duration-200 active:scale-[0.97] active:opacity-80" onclick={() => selectChatLang(1)}>
                    <div class="font-semibold text-[15px]">{language.setup.chooseChatTypeOption2}</div>
                    <div class="text-xs text-textcolor2 mt-0.5">{language.setup.chooseChatTypeOption2Desc}</div>
                </button>
                <button class="w-full px-[18px] py-4 rounded-2xl bg-darkbg text-left text-textcolor cursor-pointer transition-all duration-200 active:scale-[0.97] active:opacity-80" onclick={() => selectChatLang(2)}>
                    <div class="font-semibold text-[15px]">{language.setup.chooseChatTypeOption3}</div>
                    <div class="text-xs text-textcolor2 mt-0.5">{language.setup.chooseChatTypeOption3Desc}</div>
                </button>
            </div>
        {/if}

        <!-- Step 6+: Memory -->
        {#if step === 6 || (step > 6 && stepHistory.includes(6))}
            <div class="max-w-[75%] px-[18px] py-3.5 rounded-[20px] rounded-br-[6px] bg-borderc text-white text-sm leading-relaxed self-end animate-ob-msg">
                {language.setup[`chooseChatTypeOption${chatLang + 1}`]}
            </div>
            <div class="max-w-[85%] px-[18px] py-3.5 rounded-[20px] rounded-bl-[6px] bg-darkbg text-sm leading-relaxed self-start animate-ob-msg">
                {language.setup.chooseCheapOrMemory}
            </div>
        {/if}

        {#if step === 6}
            <div class="flex flex-col gap-2 animate-ob-msg-delay">
                <button class="w-full px-[18px] py-4 rounded-2xl bg-darkbg text-left text-textcolor cursor-pointer transition-all duration-200 active:scale-[0.97] active:opacity-80" onclick={() => selectMemory(2)}>
                    <div class="flex items-center gap-2 font-semibold text-[15px]">{language.setup.chooseCheapOrMemoryOption3} <span class="text-[10px] font-medium px-2 py-0.5 rounded-full bg-borderc/15 text-borderc">{language.recommended}</span></div>
                    <div class="text-xs text-textcolor2 mt-0.5">{language.setup.chooseCheapOrMemoryOption3Desc}</div>
                </button>
                <button class="w-full px-[18px] py-4 rounded-2xl bg-darkbg text-left text-textcolor cursor-pointer transition-all duration-200 active:scale-[0.97] active:opacity-80" onclick={() => selectMemory(0)}>
                    <div class="font-semibold text-[15px]">{language.setup.chooseCheapOrMemoryOption1}</div>
                    <div class="text-xs text-textcolor2 mt-0.5">{language.setup.chooseCheapOrMemoryOption1Desc}</div>
                </button>
                <button class="w-full px-[18px] py-4 rounded-2xl bg-darkbg text-left text-textcolor cursor-pointer transition-all duration-200 active:scale-[0.97] active:opacity-80" onclick={() => selectMemory(1)}>
                    <div class="font-semibold text-[15px]">{language.setup.chooseCheapOrMemoryOption2}</div>
                    <div class="text-xs text-textcolor2 mt-0.5">{language.setup.chooseCheapOrMemoryOption2Desc}</div>
                </button>
                <button class="w-full px-[18px] py-4 rounded-2xl bg-darkbg text-left text-textcolor cursor-pointer transition-all duration-200 active:scale-[0.97] active:opacity-80" onclick={() => selectMemory(3)}>
                    <div class="font-semibold text-[15px]">{language.setup.chooseCheapOrMemoryOption4}</div>
                    <div class="text-xs text-textcolor2 mt-0.5">{language.setup.chooseCheapOrMemoryOption4Desc}</div>
                </button>
            </div>
        {/if}

        <!-- Step 10: Done -->
        {#if step === 10}
            <div class="max-w-[85%] px-[18px] py-3.5 rounded-[20px] rounded-bl-[6px] bg-darkbg text-sm leading-relaxed self-start animate-ob-msg">
                {language.setup.allDone}
            </div>
            <div class="flex justify-center py-4 animate-ob-msg-delay">
                <div class="w-8 h-8 border-2 border-borderc border-t-transparent rounded-full animate-spin"></div>
            </div>
        {/if}
    </div>

    <!-- Input area -->
    {#if step !== 10 && step !== 0}
        <div class="shrink-0 animate-ob-msg" style="padding-bottom: env(safe-area-inset-bottom, 0px);">
            <div class="relative rounded-t-[28px] bg-darkbg ring-1 ring-darkborderc">
                <textarea
                    class="w-full pl-5 pr-14 py-4 bg-transparent text-textcolor text-[15px] outline-none resize-none overflow-y-hidden max-h-[140px] leading-normal placeholder:text-textcolor2/30"
                    bind:value={input}
                    placeholder={step === 1 ? language.setup.inputName?.replace("Lastly, i", "I")?.replace("Cuối cùng, n", "N") || "Enter your name" : step === 4 ? "sk-..." : "..."}
                    rows="1"
                    onkeydown={(e) => {
                        if (e.key === "Enter" && !e.shiftKey && !e.isComposing) {
                            e.preventDefault();
                            send();
                        }
                    }}
                    oninput={(e) => {
                        const el = e.currentTarget;
                        el.style.height = "auto";
                        el.style.height = Math.min(el.scrollHeight, 140) + "px";
                    }}
                ></textarea>
                <button
                    class="absolute bottom-3 right-3 w-10 h-10 rounded-full bg-borderc text-white flex items-center justify-center transition-all duration-300 active:scale-90 disabled:opacity-20 disabled:pointer-events-none"
                    onclick={send}
                    disabled={input.length === 0}
                >
                    <Send size={17} />
                </button>
            </div>
        </div>
    {/if}
</div>

<style>
    .ob-chat-area {
        scrollbar-width: none;
        -ms-overflow-style: none;
    }
    .ob-chat-area::-webkit-scrollbar {
        display: none;
    }

    .animate-ob-msg {
        animation: ob-msg 0.3s cubic-bezier(0.16, 1, 0.3, 1);
    }
    .animate-ob-msg-delay {
        animation: ob-msg 0.3s cubic-bezier(0.16, 1, 0.3, 1) 0.15s both;
    }

    @keyframes ob-msg {
        from { transform: translateY(12px); opacity: 0; }
        to { transform: translateY(0); opacity: 1; }
    }
</style>
