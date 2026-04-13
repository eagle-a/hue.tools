<script lang="ts">
  import ColorBlock from '$src/components/ColorBlock.svelte'
  import ColorCard from '$src/components/ColorCard.svelte'
  import { Color } from '$src/models/Color'
  import { primaryColor } from '$src/store'
  import { getColorsFromUrl, getQueryParam, updateQuery } from '$src/utils/url'
  import * as blend from 'color-blend'

  let modes = [
    { id: 'normal', name: '正常' },
    { id: 'multiply', name: '正片叠加' },
    { id: 'screen', name: '滤色' },
    { id: 'overlay', name: '叠加' },
    { id: 'darken', name: '变暗' },
    { id: 'lighten', name: '变亮' },
    { id: 'colorDodge', name: '颜色减淡' },
    { id: 'colorBurn', name: '颜色加深' },
    { id: 'hardLight', name: '强光' },
    { id: 'softLight', name: '柔光' },
    { id: 'difference', name: '差値' },
    { id: 'exclusion', name: '排除' },
    { id: 'hue', name: '色相' },
    { id: 'saturation', name: '饱和度' },
    { id: 'color', name: '颜色' },
    { id: 'luminosity', name: '亮度' },
  ]

  let mode = getQueryParam('mode', 'normal')

  $: updateQuery('mode', mode)

  let colorInstances: Color[] = getColorsFromUrl() || [
    Color.random(1),
    Color.random(1),
  ]

  // We only use the average color as the primary color for this tool.
  $: $primaryColor = new Color(
    blend.overlay(
      colorInstances[0].tinycolor.toRgb(),
      colorInstances[1].tinycolor.toRgb()
    )
  )

  $: blendedColors = modes.map((m) => ({
    mode: m.name,
    color: new Color(
      blend[m.id](...colorInstances.map((c) => c.tinycolor.toRgb()))
    ),
  }))

  const onColorChange = () => {
    updateQuery(
      'colors',
      colorInstances.map((c) => c.toString('hex8')).filter((v) => !!v)
    )
  }

  onColorChange()
</script>

<svelte:head>
  <title>hue.tools – 叠加</title>
</svelte:head>

<div class="flex-1 container mx-auto flex flex-col">
  <div
    class="flex flex-col 2xl:flex-row space-y-10 2xl:space-y-0 2xl:space-x-10"
  >
    <div class="flex flex-col">
      <h2 class="font-bold text-2xl mb-6">你的颜色</h2>
      <div
        class="flex flex-col lg:flex-row space-y-5 lg:space-x-5 lg:space-y-0 flex-1 2xl:flex-col 2xl:space-x-0 2xl:space-y-5"
      >
        {#each colorInstances as colorInstance, index (index)}
          <div class="flex-1 flex flex-col">
            <ColorCard
              deletable={colorInstances.length > 2}
              bind:color={colorInstance}
              on:change={onColorChange}
              hasTransparency
            />
          </div>
        {/each}
      </div>
    </div>

    <div class="flex-1">
      <h2 class="font-bold text-2xl mb-6">叠加模式</h2>
      <div
        class="grid gap-4 grid-cols-2 sm:grid-cols-2 md:grid-cols-3 xl:grid-cols-4 2xl:grid-cols-4"
      >
        {#each blendedColors as blendResult, index (index)}
          <div
            class="flex flex-col border dark:border-gray-700 border-gray-300 p-4 rounded-2xl"
          >
            <div class="mb-3 capitalize text-center font-medium">
              {blendResult.mode}
            </div>
            <ColorBlock
              color={blendResult.color}
              expands
              className="rounded-xl min-h-[150px]"
              showNameOnHover
            >
              <a
                on:click|stopPropagation={() => {}}
                title="查看颜色信息"
                href="/info?color={blendResult.color
                  .toString('hex8')
                  .replace('#', '')}"
                class="absolute bottom-4 right-4 opacity-0 group-hover:opacity-70 transition hover:!opacity-100"
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
        {/each}
      </div>
    </div>
  </div>
</div>
