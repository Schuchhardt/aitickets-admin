<script setup lang="ts">
interface Item {
  title: string
  icon?: string
  size?: string
  subtitle?: string
}

type Direction = 'vertical' | 'horizontal'

interface Props {
  items: Item[]
  currentStep?: number
  direction?: Direction
  iconSize?: string | number
  isActiveStepValid?: boolean
  align?: 'start' | 'center' | 'end'
}

interface Emit {
  (e: 'update:currentStep', value: number): void
}

const props = withDefaults(defineProps<Props>(), {
  currentStep: 0,
  direction: 'horizontal',
  iconSize: 52,
  isActiveStepValid: undefined,
  align: 'center',
})

const emit = defineEmits<Emit>()

const currentStep = ref(props.currentStep || 0)

// check if step is completed or active and return class name accordingly
const activeOrCompletedStepsClasses = computed(() => (index: number) => (
  index < currentStep.value
    ? 'stepper-steps-completed'
    : index === currentStep.value ? 'stepper-steps-active' : ''
))

// check if step is horizontal and not last step
const isHorizontalAndNotLastStep = computed(() => (index: number) => (
  props.direction === 'horizontal'
  && props.items.length - 1 !== index
))

// check if validation is enabled
const isValidationEnabled = computed(() => {
  return props.isActiveStepValid !== undefined
})

const getCurrentStage = (index: number, current: number) =>{
  let addedClass = ''
  if (index > current) {
    addedClass += ' future-text'
  }
  if (index == current) {
    addedClass += ' current-text'
  }
  if (index < current) {
    addedClass += ' pass-text'
  }
  return addedClass
}

watchEffect(() => {
  // we need to check undefined because if we pass 0 as currentStep it will be falsy
  if (
    props.currentStep !== undefined
    && props.currentStep < props.items.length
    && props.currentStep >= 0
  )
    currentStep.value = props.currentStep

  emit('update:currentStep', currentStep.value)
})
</script>

<template>
  <VSlideGroup
    v-model="currentStep"
    class="app-stepper"
    show-arrows
    :direction="props.direction"
    :class="`app-stepper-${props.align}`"
  >
    <VSlideGroupItem
      v-for="(item, index) in props.items"
      :key="item.title"
      :value="index"
    >
      <div
        class="cursor-pointer mx-1"
        :class="[
          (!props.isActiveStepValid && (isValidationEnabled)) && 'stepper-steps-invalid',
          activeOrCompletedStepsClasses(index),
        ]"
      >
      <!-- @click="!isValidationEnabled && emit('update:currentStep', index)" -->
        <!-- SECTION stepper step with icon -->
        <template v-if="item.icon">
          <div class="stepper-icon-step text-high-emphasis d-flex align-center gap-2" :class="getCurrentStage(index,currentStep)">
            <!-- 👉 icon and title -->
            <div
              class="d-flex align-center gap-2 step-wrapper"
              :class="[props.direction === 'horizontal' && 'flex-column']"
            >
              <div class="stepper-icon">
                <VIcon
                  :icon="item.icon"
                  :size="item.size || props.iconSize"
                />
              </div>

              <div>
                <p class="stepper-title text-base mb-0">
                  {{index + 1}}. <strong>{{ item.title }}</strong>
                </p>
                <span
                  v-if="item.subtitle"
                  class="stepper-subtitle text-sm"
                >{{ item.subtitle }}</span>
              </div>
            </div>

            <!-- 👉 append chevron -->
            <VIcon
              v-if="isHorizontalAndNotLastStep(index)"
              class="flip-in-rtl stepper-chevron-indicator mx-6"
              color="#154882"
              size="18"
              icon="mdi-chevron-right"
            />
          </div>
        </template>
        <!-- !SECTION  -->

        <!-- SECTION stepper step without icon -->
        <template v-else>
          <div class="d-flex align-center gap-x-2">
            <div class="d-flex align-center gap-2">
              <div
                class="d-flex align-center justify-center"
                style="block-size: 24px; inline-size: 24px;"
              >
                <!-- 👉 custom circle icon -->
                <template v-if="index >= currentStep">
                  <div
                    v-if="(!isValidationEnabled || props.isActiveStepValid || index !== currentStep)"
                    class="stepper-step-indicator"
                  />

                  <VIcon
                    v-else
                    icon="mdi-alert-circle-outline"
                    size="24"
                    color="error"
                  />
                </template>

                <!-- 👉 step completed icon -->

                <VIcon
                  v-else
                  icon="custom-check-circle"
                  class="stepper-step-icon"
                  size="24"
                />
              </div>

              <!-- 👉 Step Number -->
              <h4 class="text-h4 step-number">
                {{ (index + 1).toString().padStart(2, '0') }}
              </h4>
            </div>

            <!-- 👉 title and subtitle -->
            <div style="line-height: 0;">
              <h6 class="text-sm font-weight-medium step-title">
                {{ item.title }}
              </h6>

              <span
                v-if="item.subtitle"
                class="text-xs step-subtitle"
              >
                {{ item.subtitle }}
              </span>
            </div>

            <!-- 👉 stepper step line -->
            <div
              v-if="isHorizontalAndNotLastStep(index)"
              class="stepper-step-line"
            />
          </div>
        </template>
        <!-- !SECTION  -->
      </div>
    </VSlideGroupItem>
  </VSlideGroup>
</template>

<style lang="scss">
.app-stepper {
  // 👉 stepper step with bg color
  &.stepper-icon-step-bg {
    .stepper-icon-step {
      .step-wrapper {
        flex-direction: row !important;
      }

      .stepper-icon {
        display: flex;
        align-items: center;
        justify-content: center;
        border-radius: 0.3125rem;
        background-color: rgba(var(--v-theme-on-surface), var(--v-selected-opacity));
        block-size: 2.5rem;
        color: rgba(var(--v-theme-on-surface), var(--v-high-emphasis-opacity));
        inline-size: 2.5rem;
        margin-inline-end: 0.3rem;
      }

      .stepper-title,
      .stepper-subtitle {
        font-weight: 400 !important;
        line-height: normal;
      }

      .stepper-title {
        color: rgba(var(--v-theme-on-surface), var(--v-medium-emphasis-opacity));
        font-size: 0.875rem;
      }

      .stepper-subtitle {
        color: rgba(var(--v-theme-on-surface), var(--v-disabled-opacity));
        font-size: 0.75rem;
      }
    }

    .stepper-steps-active {
      .stepper-icon-step {
        .stepper-icon {
          background-color: rgb(var(--v-theme-primary));
          color: rgba(var(--v-theme-on-primary));
        }
      }
    }

    .stepper-steps-completed {
      .stepper-icon-step {
        .stepper-icon {
          background: lighten(#154882, 0.08);
          color: #154882;
        }
      }
    }
  }

  // 👉 stepper step with icon and  default
  .v-slide-group__content {
    row-gap: 1.5rem;

    .stepper-step-indicator {
      border: 0.3125rem solid rgb(var(--v-theme-primary));
      border-radius: 50%;
      background-color: rgb(var(--v-theme-surface));
      block-size: 1.25rem;
      inline-size: 1.25rem;
      opacity: var(--v-activated-opacity);
    }

    .stepper-step-line {
      border-radius: 0.1875rem;
      background-color: rgb(var(--v-theme-primary));
      block-size: 0.1875rem;
      inline-size: 3.75rem;
      opacity: var(--v-activated-opacity);
    }

    .stepper-chevron-indicator {
      color: rgba(var(--v-theme-on-surface), var(--v-disabled-opacity));
    }

    .stepper-steps-completed{
      .stepper-icon-step,
      .stepper-step-icon {
        color: #154882 !important;
      }

      .stepper-step-indicator {
        opacity: 1;
      }
    }

    .stepper-steps-active {
      .stepper-icon-step,
      .stepper-step-icon {
        // color: rgb(var(--v-theme-primary)) !important;
        color:#50C878!important;
      }

      .stepper-step-indicator {
        opacity: 1;
      }
    }

    .stepper-steps-completed {
      .stepper-step-line {
        opacity: 1;
      }

      .stepper-chevron-indicator {
        color: rgb(var(--v-theme-primary));
      }
    }

    .stepper-steps-invalid.stepper-steps-active {
      .stepper-icon-step,
      .step-number,
      .step-title,
      .step-subtitle {
        color: rgb(var(--v-theme-error)) !important;
      }
    }
  }

  // 👉 stepper alignment
  &.app-stepper-center {
    .v-slide-group__content {
      justify-content: center;
    }
  }

  &.app-stepper-start {
    .v-slide-group__content {
      justify-content: start;
    }
  }

  &.app-stepper-end {
    .v-slide-group__content {
      justify-content: end;
    }
  }
  .future-text {
    opacity: 0.5;
  }
  .current-text {
    opacity: 1;
  }
  .past-text {
    color: #154882 !important;
  }
}
</style>
