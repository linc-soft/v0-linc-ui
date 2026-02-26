<script setup lang="ts">
/**
 * NumberInput コンポーネント
 *
 * 数値入力専用のインプットコンポーネント。
 * 入力制限、範囲制御、補零、千分位区切りなどの機能を備える。
 */
import { ref, computed, watch, type InputHTMLAttributes } from 'vue'
import { cn } from '@/lib/utils'
import { ChevronUp, ChevronDown } from 'lucide-vue-next'

/** Props 定義 */
export interface NumberInputProps {
  /** v-model で使用する値 */
  modelValue?: number | null
  /** 入力位数フォーマット：「整体桁数,小数桁数」、デフォルト "9,2" */
  precision?: string
  /** 最大値、デフォルト 9999999.99 */
  max?: number
  /** 最小値、デフォルト -9999999.99 */
  min?: number
  /** 増減矢印ボタンを表示するかどうか、デフォルト false */
  showArrows?: boolean
  /** フロントエンド補零を有効にするかどうか、デフォルト false */
  padZero?: boolean
  /** 千分位区切り文字、デフォルト "," */
  separator?: string
  /** 千分位区切りを有効にするかどうか、デフォルト false */
  useSeparator?: boolean
  /** 増減ステップ値、デフォルトは小数桁数に基づいて自動計算 */
  step?: number
  /** 無効状態 */
  disabled?: boolean
  /** プレースホルダー */
  placeholder?: string
  /** 追加CSSクラス */
  class?: InputHTMLAttributes['class']
}

const props = withDefaults(defineProps<NumberInputProps>(), {
  modelValue: null,
  precision: '9,2',
  max: 9999999.99,
  min: -9999999.99,
  showArrows: false,
  padZero: false,
  separator: ',',
  useSeparator: false,
  step: undefined,
  disabled: false,
  placeholder: '',
})

const emit = defineEmits<{
  'update:modelValue': [value: number | null]
}>()

/** 精度設定をパースする */
const parsedPrecision = computed(() => {
  const parts = props.precision.split(',')
  const totalDigits = parseInt(parts[0], 10) || 9
  const decimalDigits = parseInt(parts[1], 10) || 2
  // 整数部分の最大桁数 = 整体桁数 - 小数桁数
  const integerDigits = totalDigits - decimalDigits
  return { totalDigits, decimalDigits, integerDigits }
})

/** ステップ値を計算する */
const computedStep = computed(() => {
  if (props.step !== undefined) return props.step
  // 小数桁数に応じてステップ値を自動計算
  return 1 / Math.pow(10, parsedPrecision.value.decimalDigits)
})

/** フォーカス状態 */
const isFocused = ref(false)

/** 表示用テキスト（内部状態） */
const displayText = ref('')

/**
 * 数値をフォーマットされた表示文字列に変換する
 * 補零と千分位区切りを適用
 */
function formatDisplay(value: number | null): string {
  if (value === null || value === undefined) return ''

  let str = String(value)
  const { decimalDigits } = parsedPrecision.value

  // 補零処理
  if (props.padZero && decimalDigits > 0) {
    // 小数点がなければ追加
    if (!str.includes('.')) {
      str += '.'
    }
    const parts = str.split('.')
    // 小数部分を指定桁数まで補零
    parts[1] = (parts[1] || '').padEnd(decimalDigits, '0')
    str = parts.join('.')
  }

  // 千分位区切り処理
  if (props.useSeparator && props.separator) {
    const parts = str.split('.')
    // 整数部分に千分位区切りを追加
    parts[0] = parts[0].replace(/\B(?=(\d{3})+(?!\d))/g, props.separator)
    str = parts.join('.')
  }

  return str
}

/**
 * 表示テキストから実際の数値文字列を取得する（区切り文字を除去）
 */
function stripSeparator(text: string): string {
  if (!props.useSeparator || !props.separator) return text
  // 区切り文字をすべて除去（正規表現のエスケープ処理）
  const escaped = props.separator.replace(/[.*+?^${}()|[\]\\]/g, '\\$&')
  return text.replace(new RegExp(escaped, 'g'), '')
}

/**
 * 入力文字列が有効な数値パターンかどうかを検証する
 */
function isValidInput(value: string): boolean {
  // 空文字列は許可
  if (value === '') return true
  // 負号のみは入力途中として許可
  if (value === '-') return true
  // 小数点で終わる場合は入力途中として許可
  if (value.endsWith('.')) {
    const withoutDot = value.slice(0, -1)
    return /^-?\d+$/.test(withoutDot)
  }
  // 完全な数値パターンを検証
  return /^-?\d+(\.\d+)?$/.test(value)
}

/**
 * 桁数制限を検証する
 */
function checkDigitLimit(value: string): boolean {
  if (value === '' || value === '-') return true

  const { integerDigits, decimalDigits } = parsedPrecision.value
  const isNegative = value.startsWith('-')
  const absValue = isNegative ? value.slice(1) : value

  const parts = absValue.split('.')
  const intPart = parts[0] || ''
  const decPart = parts[1] || ''

  // 整数部分の桁数チェック
  if (intPart.length > integerDigits) return false
  // 小数部分の桁数チェック
  if (decPart.length > decimalDigits) return false

  return true
}

/**
 * 数値を範囲内にクランプする
 */
function clampValue(value: number): number {
  return Math.min(props.max, Math.max(props.min, value))
}

/**
 * 入力イベントハンドラー
 * リアルタイムで入力内容を検証・制限する
 */
function onInput(event: Event) {
  const input = event.target as HTMLInputElement
  let value = input.value

  // 区切り文字を除去して純粋な数値文字列を取得
  value = stripSeparator(value)

  // 全角文字を半角に変換
  value = value.replace(/[０-９]/g, (ch) =>
    String.fromCharCode(ch.charCodeAt(0) - 0xFEE0)
  )
  value = value.replace(/．/g, '.').replace(/ー/g, '-')

  // 許可されない文字を除去（数字、負号、小数点のみ許可）
  value = value.replace(/[^0-9.\-]/g, '')

  // 負号の処理：最初の位置以外の負号を除去
  if (value.includes('-')) {
    const firstChar = value[0] === '-' ? '-' : ''
    value = firstChar + value.replace(/-/g, '')
  }

  // 小数点の処理：2つ目以降の小数点を除去
  const dotIndex = value.indexOf('.')
  if (dotIndex !== -1) {
    value = value.slice(0, dotIndex + 1) + value.slice(dotIndex + 1).replace(/\./g, '')
  }

  // 入力パターンの検証
  if (!isValidInput(value)) {
    input.value = displayText.value
    return
  }

  // 桁数制限の検証
  if (!checkDigitLimit(value)) {
    input.value = displayText.value
    return
  }

  // 表示テキストを更新
  displayText.value = value
  input.value = value

  // 有効な数値に変換できる場合はemit
  if (value === '' || value === '-') {
    if (value === '') {
      emit('update:modelValue', null)
    }
    return
  }

  // 小数点で終わる場合はまだemitしない
  if (value.endsWith('.')) return

  const numValue = parseFloat(value)
  if (!isNaN(numValue)) {
    emit('update:modelValue', numValue)
  }
}

/**
 * フォーカスイベントハンドラー
 * フォーカス時は区切り文字と補零を解除して生の数値を表示する
 */
function onFocus() {
  isFocused.value = true
  // 現在の modelValue から生の数値文字列を復元
  if (props.modelValue !== null && props.modelValue !== undefined) {
    displayText.value = String(props.modelValue)
  } else {
    displayText.value = ''
  }
}

/**
 * ブラーイベントハンドラー
 * フォーカスが外れた時に補零と千分位区切りを適用する
 */
function onBlur() {
  isFocused.value = false
  let value = stripSeparator(displayText.value)

  // 空文字列の場合
  if (value === '' || value === '-') {
    displayText.value = ''
    emit('update:modelValue', null)
    return
  }

  // 小数点で終わる場合は除去
  if (value.endsWith('.')) {
    value = value.slice(0, -1)
  }

  let numValue = parseFloat(value)
  if (isNaN(numValue)) {
    displayText.value = ''
    emit('update:modelValue', null)
    return
  }

  // 範囲制限を適用
  numValue = clampValue(numValue)

  // 小数桁数に基づいて丸める
  const { decimalDigits } = parsedPrecision.value
  numValue = parseFloat(numValue.toFixed(decimalDigits))

  emit('update:modelValue', numValue)

  // フォーマットされた表示文字列を設定
  displayText.value = formatDisplay(numValue)
}

/**
 * 値を増加させる
 */
function increment() {
  if (props.disabled) return
  const current = props.modelValue ?? 0
  const newValue = clampValue(
    parseFloat((current + computedStep.value).toFixed(parsedPrecision.value.decimalDigits))
  )
  emit('update:modelValue', newValue)
}

/**
 * 値を減少させる
 */
function decrement() {
  if (props.disabled) return
  const current = props.modelValue ?? 0
  const newValue = clampValue(
    parseFloat((current - computedStep.value).toFixed(parsedPrecision.value.decimalDigits))
  )
  emit('update:modelValue', newValue)
}

/**
 * キーダウンイベントハンドラー
 * 矢印キーによる増減をサポート
 */
function onKeyDown(event: KeyboardEvent) {
  if (event.key === 'ArrowUp') {
    event.preventDefault()
    increment()
  } else if (event.key === 'ArrowDown') {
    event.preventDefault()
    decrement()
  }
}

/**
 * modelValue の外部変更を監視して表示テキストを同期する
 */
watch(
  () => props.modelValue,
  (newVal) => {
    if (!isFocused.value) {
      // フォーカスがない場合はフォーマットされた表示を更新
      displayText.value = formatDisplay(newVal)
    }
  },
  { immediate: true }
)
</script>

<template>
  <div
    :class="cn(
      'relative flex items-center',
      props.class,
    )"
  >
    <!-- 数値入力フィールド -->
    <input
      type="text"
      inputmode="decimal"
      :value="displayText"
      :placeholder="placeholder"
      :disabled="disabled"
      :class="cn(
        'flex h-9 w-full rounded-md border border-input bg-transparent px-3 py-1 text-sm shadow-xs transition-colors',
        'file:border-0 file:bg-transparent file:text-sm file:font-medium file:text-foreground',
        'placeholder:text-muted-foreground',
        'focus-visible:outline-none focus-visible:ring-1 focus-visible:ring-ring',
        'disabled:cursor-not-allowed disabled:opacity-50',
        showArrows && 'pr-8',
      )"
      @input="onInput"
      @focus="onFocus"
      @blur="onBlur"
      @keydown="onKeyDown"
    />

    <!-- 増減矢印ボタン -->
    <div
      v-if="showArrows"
      class="absolute right-0 top-0 flex h-full flex-col border-l border-input"
    >
      <!-- 増加ボタン -->
      <button
        type="button"
        tabindex="-1"
        :disabled="disabled"
        :class="cn(
          'flex flex-1 items-center justify-center px-1 text-muted-foreground transition-colors',
          'hover:bg-accent hover:text-accent-foreground',
          'disabled:pointer-events-none disabled:opacity-50',
          'rounded-tr-md',
        )"
        @mousedown.prevent="increment"
      >
        <ChevronUp class="size-3.5" />
      </button>
      <!-- 減少ボタン -->
      <button
        type="button"
        tabindex="-1"
        :disabled="disabled"
        :class="cn(
          'flex flex-1 items-center justify-center px-1 text-muted-foreground transition-colors',
          'hover:bg-accent hover:text-accent-foreground',
          'disabled:pointer-events-none disabled:opacity-50',
          'border-t border-input rounded-br-md',
        )"
        @mousedown.prevent="decrement"
      >
        <ChevronDown class="size-3.5" />
      </button>
    </div>
  </div>
</template>
