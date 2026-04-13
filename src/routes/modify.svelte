<script lang="ts">
  import { goto } from '$app/navigation'
  import { page } from '$app/stores'

  import ColorBlock from '$src/components/ColorBlock.svelte'
  import ColorCard from '$src/components/ColorCard.svelte'
  import Modifier from '$src/components/Modifier.svelte'
  import { Color } from '$src/models/Color'
  import { primaryColor } from '$src/store'
  import { getColorFromUrl, updateQuery } from '$src/utils/url'

  let queryColor = getColorFromUrl()
  let initialFormat = queryColor ? queryColor?.tinycolor?.format : 'hex8'

  let colorInstance: Color = queryColor || Color.random()

  $: $primaryColor = colorInstance

  const onColorChange = () => {
    updateQuery('color', colorInstance.toString(initialFormat))
  }

  onColorChange()

  const modifiers = {
    brighten: 0,
    darken: 0,
    tint: 0,
    shade: 0,
    saturate: 0,
    desaturate: 0,
    lighten: 0,
    spin: 0,
  }

  const query = new URLSearchParams($page.url.search)
  for (const key in modifiers) {
    modifiers[key] = Number(query.get(key) || 0)
  }

  $: finalColor = Color.fromTinyColor(
    colorInstance.tinycolor
      .brighten(modifiers.brighten)
      .darken(modifiers.darken)
      .tint(modifiers.tint)
      .shade(modifiers.shade)
      .saturate(modifiers.saturate)
      .desaturate(modifiers.desaturate)
      .lighten(modifiers.lighten)
      .spin(modifiers.spin)
  )

  const onChange = () => {
    const query = new URLSearchParams($page.url.search)

    for (const [key, value] of Object.entries(modifiers)) {
      query.set(key, value.toString())
    }

    goto('?' + query.toString(), {
      keepfocus: true,
      replaceState: true,
      noscroll: true,
    })
  }
</script>

<svelte:head>
  <title>hue.tools – 调整</title>
</svelte:head>

<div class="flex flex-col xl:grid grid-cols-4 gap-10 container mx-auto">
  <div class="flex flex-col order-1">
    <h2 class="font-bold text-2xl mb-6">你的颜色</h2>
    <ColorCard
      deletable={false}
      {initialFormat}
      bind:color={colorInstance}
      on:change={onColorChange}
    />
  </div>

  <div class="col-span-2 my-10 xl:my-0 order-last xl:order-2">
    <h2 class="font-bold text-2xl mb-6">调整选项</h2>
    <div class="grid grid-cols-1 md:grid-cols-2 gap-5 gap-y-8">
      <Modifier
        name="提亮"
        bind:value={modifiers.brighten}
        on:change={onChange}
      >
        <div slot="description">增加亮度</div>
      </Modifier>
      <Modifier
        name="加暗"
        bind:value={modifiers.darken}
        on:change={onChange}
      >
        <div slot="description">减少亮度</div>
      </Modifier>
      <Modifier name="色调" bind:value={modifiers.tint} on:change={onChange}>
        <div slot="description">与纯白混合</div>
      </Modifier>
      <Modifier name="阴影" bind:value={modifiers.shade} on:change={onChange}>
        <div slot="description">与纯黑混合</div>
      </Modifier>
      <Modifier
        name="降饱和"
        bind:value={modifiers.desaturate}
        on:change={onChange}
      >
        <div slot="description">减少饱和度</div>
      </Modifier>
      <Modifier
        name="加饱和"
        bind:value={modifiers.saturate}
        on:change={onChange}
      >
        <div slot="description">增加饱和度</div>
      </Modifier>
      <Modifier
        name="赋亮"
        bind:value={modifiers.lighten}
        on:change={onChange}
      >
        <div slot="description">增加亮度（HSL）</div>
      </Modifier>
      <Modifier
        name="色相旋转"
        bind:value={modifiers.spin}
        on:change={onChange}
        sliderProps={{ range: { min: -360, max: 360 }, connect: false }}
        unit="°"
      >
        <div slot="description">旋转色相角度</div>
      </Modifier>
    </div>
  </div>

  <div class="flex flex-col order-3">
    <h2 class="font-bold text-2xl mb-6">结果</h2>
    <ColorBlock
      color={finalColor}
      expands
      alwaysShowColor
      size="lg"
      className="rounded-2xl min-h-[150px]"
      showName
    >
      <a
        on:click|stopPropagation={() => {}}
        href="/info?color={finalColor.hex().replace('#', '')}"
        class="absolute bottom-4 right-4 opacity-70 transition hover:opacity-100"
      >
        <svg
          xmlns="http://www.w3.org/2000/svg"
          class="h-6 w-6"
          fill="none"
          viewBox="0 0 24 24"
          stroke="currentColor"
        >
          <path
            stroke-linecap="round"
            stroke-linejoin="round"
            stroke-width="2"
            d="M13 16h-1v-4h-1m1-4h.01M21 12a9 9 0 11-18 0 9 9 0 0118 0z"
          />
        </svg>
      </a>
    </ColorBlock>
  </div>
</div>
