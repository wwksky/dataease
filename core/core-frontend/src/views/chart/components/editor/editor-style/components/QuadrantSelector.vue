<script lang="tsx" setup>
import { computed, onMounted, PropType, reactive, watch } from 'vue'
import { useI18n } from '@/hooks/web/useI18n'
import { COLOR_PANEL, DEFAULT_QUADRANT_STYLE } from '@/views/chart/components/editor/util/chart'

const { t } = useI18n()

const props = defineProps({
  chart: {
    type: Object,
    required: true
  },
  themes: {
    type: String as PropType<EditorTheme>,
    default: 'dark'
  },
  propertyInner: {
    type: Array<string>
  }
})

const predefineColors = COLOR_PANEL

const state = reactive({
  quadrantForm: JSON.parse(JSON.stringify(DEFAULT_QUADRANT_STYLE))
})
const toolTip = computed(() => {
  return props.themes === 'dark' ? 'ndark' : 'dark'
})
const emit = defineEmits(['onChangeQuadrantForm'])

watch(
  () => props.chart.customAttr.quadrant,
  () => {
    init()
  },
  { deep: true }
)

const fontSizeList = computed(() => {
  const arr = []
  for (let i = 10; i <= 40; i = i + 2) {
    arr.push({
      name: i + '',
      value: i
    })
  }
  return arr
})

const fillOpacityList = computed(() => {
  const arr = []
  for (let i = 0; i <= 1; i = i + 0.1) {
    let c = i.toFixed(1)
    arr.push({
      name: c + '',
      value: c
    })
  }
  return arr
})

const changeStyle = () => {
  emit('onChangeQuadrantForm', state.quadrantForm)
}

const init = () => {
  const chart = JSON.parse(JSON.stringify(props.chart))
  if (chart.customAttr) {
    let customAttr = null
    if (Object.prototype.toString.call(chart.customAttr) === '[object Object]') {
      customAttr = JSON.parse(JSON.stringify(chart.customAttr))
    } else {
      customAttr = JSON.parse(chart.customAttr)
    }
    if (customAttr.quadrant) {
      state.quadrantForm = customAttr.quadrant
    }
  }
}

const showProperty = prop => props.propertyInner?.includes(prop)

onMounted(() => {
  init()
})
</script>

<template>
  <el-form ref="quadrantForm" :model="state.quadrantForm" size="small" label-position="top">
    <template v-if="showProperty('lineStyle')">
      <label class="custom-form-item-label" :class="'custom-form-item-label--' + themes"
        >{{ t('chart.quadrant') }}{{ t('chart.split_line') }}</label
      >
      <div style="display: flex">
        <el-form-item class="form-item" :class="'form-item-' + themes" style="padding-right: 4px">
          <el-color-picker
            v-model="state.quadrantForm.lineStyle.stroke"
            class="color-picker-style"
            :predefine="predefineColors"
            @change="changeStyle()"
            :effect="themes"
            is-custom
          />
        </el-form-item>
        <el-form-item class="form-item" :class="'form-item-' + themes" style="padding-left: 4px">
          <el-tooltip :content="t('chart.alpha')" :effect="toolTip" placement="top">
            <el-select
              style="width: 53px"
              :effect="props.themes"
              v-model="state.quadrantForm.lineStyle.opacity"
              :placeholder="t('chart.alpha')"
              @change="changeStyle()"
            >
              <el-option
                v-for="option in fillOpacityList"
                :key="option.value"
                :label="option.name"
                :value="option.value"
              />
            </el-select>
          </el-tooltip>
        </el-form-item>
        <el-form-item class="form-item" :class="'form-item-' + themes" style="padding-left: 4px">
          <el-tooltip :content="t('chart.funnel_width')" :effect="toolTip" placement="top">
            <el-input-number
              style="width: 108px"
              :effect="props.themes"
              v-model="state.quadrantForm.lineStyle.lineWidth"
              :min="1"
              :max="10"
              size="small"
              controls-position="right"
              @change="changeStyle()"
            />
          </el-tooltip>
        </el-form-item>
      </div>
    </template>
    <div
      v-for="(l, index) in state.quadrantForm.labels"
      :key="index"
      style="flex-direction: row; justify-content: space-between"
    >
      <el-divider
        class="m-divider"
        v-if="index < state.quadrantForm.labels.length"
        :class="'m-divider--' + themes"
      />
      <label class="custom-form-item-label" :class="'custom-form-item-label--' + themes"
        >{{ t('chart.quadrant') }}{{ index + 1 }}</label
      >
      <template v-if="showProperty('regionStyle')">
        <div style="display: flex">
          <label class="custom-form-item-label" :class="'custom-form-item-label--' + themes">{{
            t('chart.backgroundColor')
          }}</label>
        </div>
        <div style="display: flex">
          <el-form-item class="form-item" :class="'form-item-' + themes" style="padding-right: 4px">
            <el-color-picker
              v-model="state.quadrantForm.regionStyle[index].fill"
              class="color-picker-style"
              :predefine="predefineColors"
              @change="changeStyle()"
              :effect="themes"
              is-custom
            />
          </el-form-item>
          <el-form-item class="form-item" :class="'form-item-' + themes" style="padding-left: 4px">
            <el-tooltip :content="t('chart.alpha')" :effect="toolTip" placement="top">
              <el-select
                style="width: 53px"
                :effect="props.themes"
                v-model="state.quadrantForm.regionStyle[index].fillOpacity"
                :placeholder="t('chart.alpha')"
                @change="changeStyle()"
              >
                <el-option
                  v-for="option in fillOpacityList"
                  :key="option.value"
                  :label="option.name"
                  :value="option.value"
                />
              </el-select>
            </el-tooltip>
          </el-form-item>
        </div>
      </template>
      <template v-if="showProperty('label')">
        <el-form-item class="form-item" :class="'form-item-' + themes" :label="t('chart.text')">
          <el-input
            :effect="props.themes"
            v-model="l.content"
            size="small"
            maxlength="50"
            @blur="changeStyle()"
          />
        </el-form-item>
        <label class="custom-form-item-label" :class="'custom-form-item-label--' + themes">
          {{ t('chart.text') }}{{ t('chart.chart_style') }}</label
        >
        <div style="display: flex">
          <el-form-item class="form-item" :class="'form-item-' + themes" style="padding-right: 4px">
            <el-color-picker
              v-model="l.style.fill"
              class="color-picker-style"
              :predefine="predefineColors"
              @change="changeStyle()"
              :effect="themes"
              is-custom
            />
          </el-form-item>
          <el-form-item class="form-item" :class="'form-item-' + themes" style="padding-left: 4px">
            <el-tooltip :content="t('chart.alpha')" :effect="toolTip" placement="top">
              <el-select
                style="width: 53px"
                :effect="props.themes"
                v-model="l.style.fillOpacity"
                :placeholder="t('chart.alpha')"
                @change="changeStyle()"
              >
                <el-option
                  v-for="option in fillOpacityList"
                  :key="option.value"
                  :label="option.name"
                  :value="option.value"
                />
              </el-select>
            </el-tooltip>
          </el-form-item>
          <el-form-item class="form-item" :class="'form-item-' + themes" style="padding-left: 4px">
            <el-tooltip :content="t('chart.font_size')" :effect="toolTip" placement="top">
              <el-select
                style="width: 108px"
                :effect="props.themes"
                v-model="l.style.fontSize"
                :placeholder="t('chart.axis_name_fontsize')"
                @change="changeStyle()"
              >
                <el-option
                  v-for="option in fontSizeList"
                  :key="option.value"
                  :label="option.name"
                  :value="option.value"
                />
              </el-select>
            </el-tooltip>
          </el-form-item>
        </div>
      </template>
    </div>
  </el-form>
</template>

<style lang="less" scoped>
.custom-form-item-label {
  margin-bottom: 4px;
  line-height: 20px;
  color: #646a73;
  font-size: 12px;
  font-style: normal;
  font-weight: 400;
  padding: 2px 12px 0 0;

  &.custom-form-item-label--dark {
    color: #a6a6a6;
  }
}

.form-item-checkbox {
  margin-bottom: 10px !important;
}

.m-divider {
  border-color: rgba(31, 35, 41, 0.15);
  margin: 0 0 16px;

  &.m-divider--dark {
    border-color: rgba(235, 235, 235, 0.15);
  }
}
</style>
