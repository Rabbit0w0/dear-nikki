<script lang="ts">
import { AUTO_MODE, DARK_MODE, LIGHT_MODE } from "@constants/constants.ts";
import I18nKey from "@i18n/i18nKey";
import { i18n } from "@i18n/translation";
import Icon from "@iconify/svelte";
import {
    applyThemeToDocument,
    getStoredTheme,
    setTheme,
} from "@utils/settings";
import { onMount } from "svelte";
import type { LIGHT_DARK_MODE } from "@/types/config.ts";

const SEQUENCE: LIGHT_DARK_MODE[] = [LIGHT_MODE, DARK_MODE, AUTO_MODE];

let mode: LIGHT_DARK_MODE = AUTO_MODE;
let panelOpen = false;

// Keep a reference so we can remove the listener on destroy
let mediaQuery: MediaQueryList | null = null;
let mediaListener: ((e: MediaQueryListEvent) => void) | null = null;

onMount(() => {
    // initialize from storage and apply immediately
    mode = getStoredTheme();
    applyThemeToDocument(mode);

    // respond to system changes ONLY when in AUTO
    mediaQuery = window.matchMedia("(prefers-color-scheme: dark)");
    mediaListener = () => {
        if (mode === AUTO_MODE) applyThemeToDocument(AUTO_MODE);
    };
    mediaQuery.addEventListener("change", mediaListener);

    return () => {
        if (mediaQuery && mediaListener) {
            mediaQuery.removeEventListener("change", mediaListener);
        }
    };
});

function switchScheme(newMode: LIGHT_DARK_MODE) {
    mode = newMode;
    setTheme(newMode);
    applyThemeToDocument(newMode);
}

function toggleScheme() {
    const idx = SEQUENCE.indexOf(mode);
    switchScheme(SEQUENCE[(idx + 1) % SEQUENCE.length]);
}

function openPanel() {
    panelOpen = true;
}
function closePanel() {
    panelOpen = false;
}

function onKeyDown(e: KeyboardEvent) {
    if (e.key === "Escape") {
        closePanel();
        // move focus back to the trigger button for accessibility
        const btn = document.getElementById("scheme-switch");
        btn?.focus();
    }
}
</script>

<!-- z-50 keeps this above other float panels -->
<div
    class="relative z-50"
    role="menu"
    tabindex="-1"
    on:mouseleave={closePanel}
    on:keydown={onKeyDown}
>
    <button
        id="scheme-switch"
        class="relative btn-plain scale-animation rounded-[var(--radius-large)] h-11 w-11 active:scale-90"
        aria-label="Light/Dark Mode"
        aria-haspopup="menu"
        aria-expanded={panelOpen}
        role="menuitem"
        on:click={toggleScheme}
        on:mouseenter={openPanel}
        on:focusin={openPanel}
    >
        <!-- Light icon -->
        <div class="absolute" class:opacity-0={mode !== LIGHT_MODE}>
            <Icon
                icon="material-symbols:wb-sunny-outline-rounded"
                class="text-[1.25rem]"
            />
        </div>
        <!-- Dark icon -->
        <div class="absolute" class:opacity-0={mode !== DARK_MODE}>
            <Icon
                icon="material-symbols:dark-mode-outline-rounded"
                class="text-[1.25rem]"
            />
        </div>
        <!-- Auto icon -->
        <div class="absolute" class:opacity-0={mode !== AUTO_MODE}>
            <Icon
                icon="material-symbols:radio-button-partial-outline"
                class="text-[1.25rem]"
            />
        </div>
    </button>

    <!-- Panel -->
    <div
        id="light-dark-panel"
        class="hidden lg:block absolute transition top-11 -right-2 pt-5"
        class:float-panel-closed={!panelOpen}
        on:focusout={(e) => {
            // Close when focus leaves the panel (and not moving to a child)
            const related = (e as FocusEvent).relatedTarget as Node | null;
            if (!e.currentTarget.contains(related)) closePanel();
        }}
    >
        <div class="card-base float-panel p-2">
            <button
                class="flex transition whitespace-nowrap items-center !justify-start w-full btn-plain scale-animation rounded-[var(--radius-large)] h-9 px-3 font-medium active:scale-95 mb-0.5"
                class:current-theme-btn={mode === LIGHT_MODE}
                on:click={() => switchScheme(LIGHT_MODE)}
                role="menuitemradio"
                aria-checked={mode === LIGHT_MODE}
            >
                <Icon
                    icon="material-symbols:wb-sunny-outline-rounded"
                    class="text-[1.25rem] mr-3"
                />
                {i18n(I18nKey.lightMode)}
            </button>

            <button
                class="flex transition whitespace-nowrap items-center !justify-start w-full btn-plain scale-animation rounded-[var(--radius-large)] h-9 px-3 font-medium active:scale-95 mb-0.5"
                class:current-theme-btn={mode === DARK_MODE}
                on:click={() => switchScheme(DARK_MODE)}
                role="menuitemradio"
                aria-checked={mode === DARK_MODE}
            >
                <Icon
                    icon="material-symbols:dark-mode-outline-rounded"
                    class="text-[1.25rem] mr-3"
                />
                {i18n(I18nKey.darkMode)}
            </button>

            <button
                class="flex transition whitespace-nowrap items-center !justify-start w-full btn-plain scale-animation rounded-[var(--radius-large)] h-9 px-3 font-medium active:scale-95"
                class:current-theme-btn={mode === AUTO_MODE}
                on:click={() => switchScheme(AUTO_MODE)}
                role="menuitemradio"
                aria-checked={mode === AUTO_MODE}
            >
                <Icon
                    icon="material-symbols:radio-button-partial-outline"
                    class="text-[1.25rem] mr-3"
                />
                {i18n(I18nKey.systemMode)}
            </button>
        </div>
    </div>
</div>
