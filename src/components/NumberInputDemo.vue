<script setup lang="ts">
/**
 * NumberInput デモコンポーネント
 * NumberInput の各機能を展示する
 */
import { ref } from 'vue'
import NumberInput from '@/components/ui/NumberInput.vue'

/** デモ用のリアクティブ値 */
const basicValue = ref<number | null>(null)
const arrowValue = ref<number | null>(12345.67)
const padZeroValue = ref<number | null>(42.5)
const separatorValue = ref<number | null>(1234567.89)
const rangeValue = ref<number | null>(50)
const precisionValue = ref<number | null>(123.456)
const allFeaturesValue = ref<number | null>(9876543.21)
</script>

<template>
  <div class="mx-auto max-w-2xl space-y-8 p-8">
    <div>
      <h1 class="text-2xl font-bold text-foreground">NumberInput 组件演示</h1>
      <p class="mt-1 text-sm text-muted-foreground">
        数值输入专用组件，支持输入限制、范围控制、补零、千分位分隔等功能。
      </p>
    </div>

    <!-- 基本用法 -->
    <section class="space-y-3 rounded-lg border border-border p-5">
      <div>
        <h2 class="text-base font-semibold text-foreground">基本用法</h2>
        <p class="text-sm text-muted-foreground">默认配置：精度 9,2（整体9位数字，小数2位）</p>
      </div>
      <NumberInput
        v-model="basicValue"
        placeholder="请输入数值..."
      />
      <p class="text-xs text-muted-foreground">
        当前值：<code class="rounded bg-muted px-1.5 py-0.5 font-mono text-foreground">{{ basicValue ?? 'null' }}</code>
      </p>
    </section>

    <!-- 增减箭头 -->
    <section class="space-y-3 rounded-lg border border-border p-5">
      <div>
        <h2 class="text-base font-semibold text-foreground">增减箭头</h2>
        <p class="text-sm text-muted-foreground">启用 showArrows 属性后，右侧会显示增减按钮。也可以用键盘上下箭头操作。</p>
      </div>
      <NumberInput
        v-model="arrowValue"
        :show-arrows="true"
        placeholder="带增减箭头"
      />
      <p class="text-xs text-muted-foreground">
        当前值：<code class="rounded bg-muted px-1.5 py-0.5 font-mono text-foreground">{{ arrowValue ?? 'null' }}</code>
      </p>
    </section>

    <!-- 前端补零 -->
    <section class="space-y-3 rounded-lg border border-border p-5">
      <div>
        <h2 class="text-base font-semibold text-foreground">前端补零</h2>
        <p class="text-sm text-muted-foreground">
          启用 padZero 属性后，失焦时自动补齐小数位数的零。例如 42.5 会显示为 42.50。
        </p>
      </div>
      <NumberInput
        v-model="padZeroValue"
        :pad-zero="true"
        placeholder="失焦后补零"
      />
      <p class="text-xs text-muted-foreground">
        当前值：<code class="rounded bg-muted px-1.5 py-0.5 font-mono text-foreground">{{ padZeroValue ?? 'null' }}</code>
      </p>
    </section>

    <!-- 千分位分隔 -->
    <section class="space-y-3 rounded-lg border border-border p-5">
      <div>
        <h2 class="text-base font-semibold text-foreground">千分位分隔</h2>
        <p class="text-sm text-muted-foreground">
          启用 useSeparator 属性后，失焦时自动添加千分位分隔符。例如 1234567.89 会显示为 1,234,567.89。
        </p>
      </div>
      <NumberInput
        v-model="separatorValue"
        :use-separator="true"
        placeholder="失焦后千分位分隔"
      />
      <p class="text-xs text-muted-foreground">
        当前值：<code class="rounded bg-muted px-1.5 py-0.5 font-mono text-foreground">{{ separatorValue ?? 'null' }}</code>
      </p>
    </section>

    <!-- 范围限制 -->
    <section class="space-y-3 rounded-lg border border-border p-5">
      <div>
        <h2 class="text-base font-semibold text-foreground">范围限制</h2>
        <p class="text-sm text-muted-foreground">
          通过 min 和 max 属性限制数值范围。此示例限制在 0~100 之间，配合增减箭头使用。
        </p>
      </div>
      <NumberInput
        v-model="rangeValue"
        :min="0"
        :max="100"
        :show-arrows="true"
        :step="1"
        placeholder="0~100"
      />
      <p class="text-xs text-muted-foreground">
        当前值：<code class="rounded bg-muted px-1.5 py-0.5 font-mono text-foreground">{{ rangeValue ?? 'null' }}</code>
      </p>
    </section>

    <!-- 自定义精度 -->
    <section class="space-y-3 rounded-lg border border-border p-5">
      <div>
        <h2 class="text-base font-semibold text-foreground">自定义精度</h2>
        <p class="text-sm text-muted-foreground">
          精度设置为 10,3（整体10位，小数3位），并启用补零功能。
        </p>
      </div>
      <NumberInput
        v-model="precisionValue"
        precision="10,3"
        :pad-zero="true"
        :show-arrows="true"
        placeholder="最多7位整数 + 3位小数"
      />
      <p class="text-xs text-muted-foreground">
        当前值：<code class="rounded bg-muted px-1.5 py-0.5 font-mono text-foreground">{{ precisionValue ?? 'null' }}</code>
      </p>
    </section>

    <!-- 全功能组合 -->
    <section class="space-y-3 rounded-lg border border-border p-5">
      <div>
        <h2 class="text-base font-semibold text-foreground">全功能组合</h2>
        <p class="text-sm text-muted-foreground">
          同时启用增减箭头、补零和千分位分隔功能。
        </p>
      </div>
      <NumberInput
        v-model="allFeaturesValue"
        :show-arrows="true"
        :pad-zero="true"
        :use-separator="true"
        placeholder="全功能演示"
      />
      <p class="text-xs text-muted-foreground">
        当前值：<code class="rounded bg-muted px-1.5 py-0.5 font-mono text-foreground">{{ allFeaturesValue ?? 'null' }}</code>
      </p>
    </section>

    <!-- 禁用状态 -->
    <section class="space-y-3 rounded-lg border border-border p-5">
      <div>
        <h2 class="text-base font-semibold text-foreground">禁用状态</h2>
        <p class="text-sm text-muted-foreground">设置 disabled 属性后，输入框不可编辑。</p>
      </div>
      <NumberInput
        :model-value="999.99"
        :disabled="true"
        :show-arrows="true"
        :pad-zero="true"
        :use-separator="true"
      />
    </section>
  </div>
</template>
