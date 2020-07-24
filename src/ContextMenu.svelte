<script>
    // Version 0.4.1
    import { setContext as baseSetContext, onMount, onDestroy } from 'svelte';
    import { fade } from 'svelte/transition';
    import { preventScrolling, isAncestorOfNode, disableDefaultContextMenu } from './utils';
    import Menu from './Menu';

    export let key = 'simple-context-menu';
    export let setContext = baseSetContext;
    export let transitionBg = fade;
    export let transitionBgProps = { duration: 250 };
    
    let _pageX, _pageY, _items = [], _closeOnAction = true, _openDirection;

    const toVoid = () => {};
    let onOpen = toVoid;
    let onClose = toVoid;
    let onOpened = toVoid;
    let onClosed = toVoid;

    let enableScroll;

    export const open = (
        {pageX, pageY, closeOnAction = true, openDirection} = {},
        items = [],
        callback = {}
    ) => {
        console.log('ContextMenu/open', pageX, pageX, items)
        _items = items,
        _pageX = pageX;
        _pageY = pageY;
        _closeOnAction = closeOnAction;
        _openDirection = openDirection;
        onOpen = callback.onOpen || toVoid;
        onClose = callback.onClose || toVoid;
        onOpened = callback.onOpened || toVoid;
        onClosed = callback.onClosed || toVoid;
        enableScroll = preventScrolling();
    };

    export const close = (callback = {}) => {
        onClose = callback.onClose || onClose;
        onClosed = callback.onClosed || onClosed;
        _pageX = undefined;
        _pageY = undefined;
        _items = [];
        // closeSubmenu();

        enableScroll = enableScroll && enableScroll();
    };

    setContext(key, { open, close });

    let contentElement;
    let enableDefaultContextMenu;
    // onMount(async () => enableDefaultContextMenu = disableDefaultContextMenu(contentElement));
    onDestroy(async () => {
        // enableDefaultContextMenu && enableDefaultContextMenu();
        enableScroll = enableScroll && enableScroll();
    });


    let windowW, windowH, menuW, menuH;

    const calcMenuDimentions = (pageX, pageY, windowH, windowW, menuH, menuW) => {
        console.log('calcMenuDimentions', pageX, pageY, windowW, windowH)
        if (pageX === undefined || pageY === undefined) return;
        if (_openDirection === 'left') {
            return {
                top: pageY,
                left: (pageX - menuW),
            };
        } else if (_openDirection === 'right') {
            return {
                top: pageY,
                left: pageX,
            };
        } else {
            return {
                top: pageY,
                left: pageX < (windowW / 2) ? pageX : (pageX - menuW),
            };
        }
    };

    $: menu = calcMenuDimentions(_pageX, _pageY, windowH, windowW, menuH, menuW);

    let container;

    const preventDefault = e => {
        if (isAncestorOfNode(contentElement, event.target) || isAncestorOfNode(container, event.target)) {
            e.preventDefault();
        }
    };

    const handleBgMouseup = event => {
        if (menu !== undefined && !isAncestorOfNode(container, event.target)) {
            close();
        }
    };

    const actionPerformed = () => {
        _closeOnAction && close();
    };

</script>

<svelte:window bind:outerWidth={windowW} 
    bind:outerHeight={windowH} 
    on:mousedown={handleBgMouseup}
    on:touchstart={handleBgMouseup}
    on:contextmenu={preventDefault}
/>

{#if menu !== undefined}
<div class="container" bind:this={container}
    transition:transitionBg={transitionBgProps}
    style="--menu-left:{menu.left}px; --menu-top:{menu.top}px" 
    bind:offsetWidth={menuW}
    bind:offsetHeight={menuH}>
    <Menu items={_items} on:performed={actionPerformed}/>
</div>
{/if}

<div bind:this={contentElement} on:contextmenu={preventDefault}>
    <slot></slot>
</div>

<style>
    .container {
        position: fixed;
        top: var(--menu-top);
        left: var(--menu-left);
        z-index: 1000;

        min-width: 5rem;
        color: black;
        background: white;
        box-shadow: 0 4px 5px 3px rgba(0, 0, 0, 0.2);
        border-radius: 0.5rem;
    }
</style>