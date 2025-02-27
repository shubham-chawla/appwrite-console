<script lang="ts">
    import { registerTheme, use, init } from 'echarts/core';
    import { onDestroy, onMount } from 'svelte';
    import { app } from '$lib/stores/app';
    import { defaultConfig } from './config';
    import base from './base.json';
    import light from './light.json';
    import dark from './dark.json';
    import type { ECharts } from 'echarts/core';
    import type { BarSeriesOption, LineSeriesOption } from 'echarts/charts';
    import type { EChartsOption } from 'echarts';

    registerTheme('light', { ...base, ...light });
    registerTheme('dark', { ...base, ...dark });

    export let options: EChartsOption;
    export let series: (BarSeriesOption | LineSeriesOption)[];

    let chart: ECharts;
    let container: HTMLDivElement;

    $: option = {
        ...defaultConfig,
        ...options,
        series
    };

    $: option && setOption();
    $: if (chart && $app.themeInUse) {
        makeChart();
        setOption();
    }

    onMount(async () => {
        use([
            (await import('echarts/charts')).BarChart,
            (await import('echarts/charts')).LineChart,
            (await import('echarts/components')).TitleComponent,
            (await import('echarts/components')).TooltipComponent,
            (await import('echarts/components')).GridComponent,
            (await import('echarts/components')).DatasetComponent,
            (await import('echarts/components')).TransformComponent,
            (await import('echarts/components')).LegendComponent,
            (await import('echarts/features')).LabelLayout,
            (await import('echarts/renderers')).SVGRenderer
        ]);
        makeChart();
    });

    onDestroy(() => {
        destroyChart();
    });

    const setOption = () => {
        if (chart && !chart.isDisposed()) {
            chart.setOption(option);
        }
    };

    const destroyChart = () => {
        if (chart && !chart.isDisposed()) {
            chart.dispose();
        }
    };

    const makeChart = () => {
        destroyChart();
        chart = init(container, $app.themeInUse);
    };

    let timeoutId: unknown;
    const onResize = () => {
        if (timeoutId == undefined) {
            timeoutId = setTimeout(() => {
                if (chart && !chart.isDisposed()) {
                    chart.resize();
                }
                timeoutId = undefined;
            }, 250);
        }
    };
</script>

<svelte:window on:resize={onResize} />

<div class="echart" bind:this={container} />

<style>
    .echart {
        width: 100%;
        min-height: 10rem;
        height: 100%;
    }
</style>
