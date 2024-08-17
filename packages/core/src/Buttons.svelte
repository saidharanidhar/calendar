<script>
    import { getContext } from "svelte";
    import {
        createDate,
        cloneDate,
        subtractDay,
        addDuration,
        setContent,
        subtractDuration,
        setMidnight,
    } from "./lib.js";

    export let buttons;

    let {
        _currentRange,
        _viewTitle,
        buttonText,
        customButtons,
        date,
        duration,
        hiddenDays,
        theme,
        view,
        viewDropDownOptions,
        resourceToggleChecked,
    } = getContext("state");

    let today = setMidnight(createDate()),
        isToday;

    $: isToday =
        (today >= $_currentRange.start && today < $_currentRange.end) || null;

    function prev() {
        let d = subtractDuration($date, $duration);
        if ($hiddenDays.length && $hiddenDays.length < 7) {
            while ($hiddenDays.includes(d.getUTCDay())) {
                subtractDay(d);
            }
        }
        $date = d;
    }

    function next() {
        $date = addDuration($date, $duration);
    }

    $: dropDownVisibility =
        $viewDropDownOptions.length > 0 ? "visible" : "hidden";

    $: resourceToggleDisabled = ![
        "timeGridDay",
        "timeGridWeek",
        "resourceTimeGridDay",
        "resourceTimeGridWeek",
    ].includes($view);
</script>

{#each buttons as button}
    {#if button == "title"}
        <!-- svelte-ignore a11y-missing-content -->
        <h2 class={$theme.title} use:setContent={$_viewTitle}></h2>
    {:else if button == "prev"}
        <button
            class="{$theme.button} ec-{button}"
            aria-label={$buttonText.prev}
            title={$buttonText.prev}
            on:click={prev}><i class="{$theme.icon} ec-{button}"></i></button
        >
    {:else if button == "next"}
        <button
            class="{$theme.button} ec-{button}"
            aria-label={$buttonText.next}
            title={$buttonText.next}
            on:click={next}><i class="{$theme.icon} ec-{button}"></i></button
        >
    {:else if button == "today"}
        <button
            class="{$theme.button} ec-{button}"
            on:click={() => ($date = cloneDate(today))}
            disabled={isToday}>{$buttonText[button]}</button
        >
    {:else if button == "viewDropDown"}
        <select
            class="{$theme.button} ec-{button}"
            bind:value={$view}
            style="visibility: {dropDownVisibility};"
        >
            {#each $viewDropDownOptions as key (key)}
                <option value={key}>{$buttonText[key]}</option>
            {/each}
        </select>
    {:else if $customButtons[button]}
        <button
            class="{$theme.button} ec-{button}"
            on:click={$customButtons[button].click}
            >{$customButtons[button].text}</button
        >
    {:else if button == "resourceToggle"}
        <label class="ec-switch" title="Resource View">
            <input
                type="checkbox"
                on:change={(event) => {
                    if (event.target.checked) {
                        if ($view === "timeGridDay") {
                            $view = "resourceTimeGridDay";
                        } else if ($view === "timeGridWeek") {
                            $view = "resourceTimeGridWeek";
                        }
                        $resourceToggleChecked = true;
                    } else {
                        if ($view === "resourceTimeGridDay") {
                            $view = "timeGridDay";
                        } else if ($view === "resourceTimeGridWeek") {
                            $view = "timeGridWeek";
                        }
                        $resourceToggleChecked = false;
                    }
                }}
                disabled={resourceToggleDisabled}
                checked={$resourceToggleChecked}
            />
            <span class="slider round"></span>
        </label>
    {:else if button != ""}
        <button
            class="{$theme.button}{($resourceToggleChecked &&
                $view === 'resourceTimeGridDay' &&
                button === 'timeGridDay') ||
            ($resourceToggleChecked &&
                $view === 'resourceTimeGridWeek' &&
                button === 'timeGridWeek') ||
            $view === button
                ? ' ' + $theme.active
                : ''} ec-{button}"
            on:click={() => {
                if ($resourceToggleChecked && button == "timeGridDay") {
                    $view = "resourceTimeGridDay";
                } else if ($resourceToggleChecked && button == "timeGridWeek") {
                    $view = "resourceTimeGridWeek";
                } else {
                    $view = button;
                    $resourceToggleChecked = false;
                }
                if (button != "timeGridDay" && button != "timeGridWeek") {
                    $resourceToggleChecked = false;
                }
            }}>{$buttonText[button]}</button
        >
    {/if}
{/each}
