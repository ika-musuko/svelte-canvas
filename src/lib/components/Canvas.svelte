<script context="module" lang="ts">
  import { getContext as getCtx } from 'svelte';
  import LayerManager from '../util/LayerManager';

  const KEY = Symbol();
  export const getRegisterLayer = (): LayerManager['register'] => getCtx(KEY);
</script>

<script lang="ts">
  import type { ResizeEvent } from '../types';
  import {
    createHitCanvas,
    type HitCanvasRenderingContext2D,
  } from 'hit-canvas';
  import {
    onMount,
    onDestroy,
    setContext,
    createEventDispatcher,
  } from 'svelte';

  export let width: number | null = null,
    height: number | null = null,
    pixelRatio: number | null = null,
    style = '',
    autoplay = false,
    autoclear = true,
    layerEvents = false;

  let canvas: HTMLCanvasElement;
  let context: CanvasRenderingContext2D | HitCanvasRenderingContext2D | null =
    null;
  let layerRef: HTMLDivElement;

  let devicePixelRatio: number | undefined;
  let className = '';

  let canvasWidth: number;
  let canvasHeight: number;

  export { className as class, redraw, getCanvas, getContext };

  const dispatch = createEventDispatcher<ResizeEvent>();

  const manager = new LayerManager();
  setContext(KEY, manager.register);

  function redraw() {
    manager.redraw();
  }

  function getCanvas() {
    return canvas;
  }

  function getContext() {
    return <CanvasRenderingContext2D>context;
  }

  onMount(() => {
    if (layerEvents) {
      context = createHitCanvas(canvas);
      manager.onRenderingLayerIdChange((id) => {
        (<HitCanvasRenderingContext2D>context).setCurrentLayerId(id);
      });
    } else {
      context = canvas.getContext('2d');
    }

    manager.init(<CanvasRenderingContext2D>context, layerRef);
  });

  onDestroy(() => manager.destroy());

  const resize = (node: Element) => {
    const canvasObserver = new ResizeObserver(([{ contentRect }]) => {
      canvasWidth = contentRect.width;
      canvasHeight = contentRect.height;
    });

    canvasObserver.observe(node);

    return {
      destroy: () => canvasObserver.disconnect(),
    };
  };

  const handleLayerMouseMove = (e: MouseEvent) => {
    const x = e.offsetX * _pixelRatio;
    const y = e.offsetY * _pixelRatio;
    const id = (<HitCanvasRenderingContext2D>context).getLayerIdAt(x, y);
    manager.setActiveLayer(id, e);
    manager.dispatchLayerEvent(e);
  };

  const handleLayerTouchStart = (e: TouchEvent) => {
    const { clientX, clientY } = e.changedTouches[0];
    const { left, top } = canvas.getBoundingClientRect();
    const x = (clientX - left) * _pixelRatio;
    const y = (clientY - top) * _pixelRatio;
    const id = (<HitCanvasRenderingContext2D>context).getLayerIdAt(x, y);
    manager.setActiveLayer(id, e);
    manager.dispatchLayerEvent(e);
  };

  const handleLayerEvent = (e: MouseEvent | TouchEvent) => {
    manager.dispatchLayerEvent(e);
  };

  $: _width = width ?? canvasWidth ?? 0;
  $: _height = height ?? canvasHeight ?? 0;
  $: _pixelRatio = pixelRatio ?? devicePixelRatio ?? 2;

  $: manager.width = _width;
  $: manager.height = _height;
  $: manager.pixelRatio = _pixelRatio;
  $: manager.autoplay = autoplay;
  $: manager.autoclear = autoclear;

  $: _width, _height, _pixelRatio, autoclear, manager.redraw();

  $: layerMouseMoveHandler = layerEvents ? handleLayerMouseMove : null;
  $: layerTouchStartHandler = layerEvents ? handleLayerTouchStart : null;
  $: layerEventHandler = layerEvents ? handleLayerEvent : null;

  $: dispatch('resize', { width: _width, height: _height });
</script>

<svelte:window bind:devicePixelRatio />

<canvas
  bind:this={canvas}
  use:resize
  bind:clientWidth={canvasWidth}
  bind:clientHeight={canvasHeight}
  class={className}
  width={_width * _pixelRatio}
  height={_height * _pixelRatio}
  style={style || `width: ${width ? `${width}px` : '100%'}; height: ${height ? `${height}px` : '100%'}`}
  on:touchstart|preventDefault={layerTouchStartHandler}
  on:mousemove={layerMouseMoveHandler}
  on:pointermove={layerMouseMoveHandler}
  on:click={layerEventHandler}
  on:contextmenu={layerEventHandler}
  on:dblclick={layerEventHandler}
  on:mousedown={layerEventHandler}
  on:mouseenter={layerEventHandler}
  on:mouseleave={layerEventHandler}
  on:mouseup={layerEventHandler}
  on:wheel={layerEventHandler}
  on:touchcancel|preventDefault={layerEventHandler}
  on:touchend|preventDefault={layerEventHandler}
  on:touchmove|preventDefault={layerEventHandler}
  on:pointerenter={layerEventHandler}
  on:pointerleave={layerEventHandler}
  on:pointerdown={layerEventHandler}
  on:pointerup={layerEventHandler}
  on:pointercancel={layerEventHandler}
  on:focus
  on:blur
  on:fullscreenchange
  on:fullscreenerror
  on:scroll
  on:cut
  on:copy
  on:paste
  on:keydown
  on:keypress
  on:keyup
  on:auxclick
  on:click
  on:contextmenu
  on:dblclick
  on:mousedown
  on:mouseenter
  on:mouseleave
  on:mousemove
  on:mouseover
  on:mouseout
  on:mouseup
  on:select
  on:wheel
  on:drag
  on:dragend
  on:dragenter
  on:dragstart
  on:dragleave
  on:dragover
  on:drop
  on:touchcancel
  on:touchend
  on:touchmove
  on:touchstart
  on:pointerover
  on:pointerenter
  on:pointerdown
  on:pointermove
  on:pointerup
  on:pointercancel
  on:pointerout
  on:pointerleave
  on:gotpointercapture
  on:lostpointercapture
/>

<div style:display="none" bind:this={layerRef}>
  <slot />
</div>
