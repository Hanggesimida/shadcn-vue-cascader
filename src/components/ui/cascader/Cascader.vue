<script setup lang="ts">
import { ref, computed, watch, nextTick, type HTMLAttributes } from "vue"
import { ChevronRight, ChevronDown, X, ArrowLeft } from "lucide-vue-next"
import { cn } from "@/lib/utils"
import {
    Popover,
    PopoverTrigger,
    PopoverContent,
} from "@/components/ui/popover"
import {
    Drawer,
    DrawerContent,
    DrawerHeader,
    DrawerTitle,
    DrawerTrigger,
} from "@/components/ui/drawer"
import { useBreakpoints } from '@vueuse/core'

// ============================================================================
// Type Definitions
// ============================================================================

export interface CascaderOption {
    value: string
    label: string | any
    textLabel?: string
    disabled?: boolean
    children?: CascaderOption[]
}

// ============================================================================
// Props & Emits
// ============================================================================

const props = defineProps<{
    options: CascaderOption[]
    modelValue?: string[]
    placeholder?: string
    disabled?: boolean
    allowClear?: boolean
    expandTrigger?: "click" | "hover"
    class?: HTMLAttributes["class"]
}>()

const emit = defineEmits<{
    (e: "update:modelValue", value: string[]): void
    (e: "change", value: string[], options: CascaderOption[]): void
}>()

// ============================================================================
// Responsive Breakpoints
// ============================================================================

const breakpoints = useBreakpoints({
    sm: 640,
    md: 768,
    lg: 1024,
    xl: 1280,
})
const isMobile = breakpoints.smaller('md')

// ============================================================================
// State Management
// ============================================================================

const open = ref(false)
const internalValue = ref<string[]>([])
const expandedPath = ref<string[]>([])
const focusedColumn = ref(0)
const focusedIndex = ref(0)

// Mobile state
const mobileActiveLevel = ref(0)
const mobilePath = ref<string[]>([])

// ============================================================================
// Computed Properties - Desktop
// ============================================================================

const selectedValue = computed(() =>
    props.modelValue ?? internalValue.value
)

function getLabel(opt: CascaderOption) {
    return opt.textLabel || opt.label || opt.value
}

/** Generate columns based on expandedPath */
const columns = computed(() => {
    const cols: CascaderOption[][] = [props.options]
    let current = props.options

    for (const val of expandedPath.value) {
        const found = current.find(o => o.value === val)
        if (found?.children) {
            cols.push(found.children)
            current = found.children
        } else {
            break
        }
    }
    return cols
})

function getSelectedOptions(path: string[]) {
    const result: CascaderOption[] = []
    let current = props.options

    for (const val of path) {
        const found = current.find(o => o.value === val)
        if (!found) break
        result.push(found)
        current = found.children || []
    }
    return result
}

const selectedOptions = computed(() =>
    getSelectedOptions(selectedValue.value)
)

const displayValue = computed(() => {
    if (!selectedOptions.value.length) return null
    return selectedOptions.value.map(o => getLabel(o)).join(" / ")
})

// ============================================================================
// Computed Properties - Mobile
// ============================================================================

/** Get current level options for mobile */
const mobileCurrentColumn = computed(() => {
    if (mobileActiveLevel.value === 0) {
        return props.options
    }
    let current = props.options
    for (let i = 0; i < mobileActiveLevel.value; i++) {
        const val = mobilePath.value[i]
        const found = current.find(o => o.value === val)
        if (found?.children) {
            current = found.children
        } else {
            return []
        }
    }
    return current
})

/** Get current level title for mobile */
const mobileCurrentTitle = computed(() => {
    if (mobileActiveLevel.value === 0) {
        return props.placeholder || 'Please select'
    }
    let current = props.options
    for (let i = 0; i < mobileActiveLevel.value; i++) {
        const val = mobilePath.value[i]
        const found = current.find(o => o.value === val)
        if (!found) return props.placeholder || 'Please select'
        if (i === mobileActiveLevel.value - 1) {
            return getLabel(found)
        }
        if (found.children) {
            current = found.children
        } else {
            return props.placeholder || 'Please select'
        }
    }
    return props.placeholder || 'Please select'
})

// ============================================================================
// Event Handlers - Desktop
// ============================================================================

function selectOption(option: CascaderOption, columnIndex: number) {
    if (option.disabled) return

    const newPath = [
        ...expandedPath.value.slice(0, columnIndex),
        option.value,
    ]

    if (option.children?.length) {
        expandedPath.value = newPath
        focusedColumn.value = columnIndex + 1
        focusedIndex.value = 0
    } else {
        internalValue.value = newPath
        emit("update:modelValue", newPath)
        emit("change", newPath, getSelectedOptions(newPath))
        open.value = false
        expandedPath.value = []
    }
}

function clear(e: MouseEvent) {
    e.stopPropagation()
    internalValue.value = []
    emit("update:modelValue", [])
    emit("change", [], [])
    open.value = false
}

// ============================================================================
// Event Handlers - Mobile
// ============================================================================

/** Handle mobile selection */
function handleMobileSelect(opt: CascaderOption) {
    if (opt.disabled) return

    if (opt.children?.length) {
        // Has children: navigate to next level
        mobilePath.value = [
            ...mobilePath.value.slice(0, mobileActiveLevel.value),
            opt.value
        ]
        mobileActiveLevel.value++
    } else {
        // No children: selection complete
        const finalPath = [
            ...mobilePath.value.slice(0, mobileActiveLevel.value),
            opt.value
        ]
        internalValue.value = finalPath
        emit("update:modelValue", finalPath)
        emit("change", finalPath, getSelectedOptions(finalPath))
        open.value = false
    }
}

// ============================================================================
// Watchers
// ============================================================================

watch(open, async (val) => {
    if (val) {
        if (isMobile.value) {
            // Mobile: reset to first level
            mobileActiveLevel.value = 0
            mobilePath.value = []
        } else {
            // Desktop: expand selected path
            expandedPath.value = selectedValue.value.slice(0, -1)
            focusedColumn.value = 0
            focusedIndex.value = 0
        }
        await nextTick()
    } else {
        expandedPath.value = []
        mobileActiveLevel.value = 0
        mobilePath.value = []
    }
})

// ============================================================================
// Styles
// ============================================================================

const inputBaseClass =
    'file:text-foreground placeholder:text-muted-foreground ' +
    'selection:bg-primary selection:text-primary-foreground ' +
    'dark:bg-input/30 border-input h-9 w-full min-w-0 rounded-md border ' +
    'bg-transparent px-3 py-1 text-base shadow-xs outline-none ' +
    'transition-[color,box-shadow] ' +
    'focus-visible:border-ring focus-visible:ring-ring/50 focus-visible:ring-[3px] ' +
    'disabled:pointer-events-none disabled:cursor-not-allowed disabled:opacity-50 ' +
    'md:text-sm'

</script>

<template>
    <component :is="isMobile ? Drawer : Popover" v-model:open="open">
        <component :is="isMobile ? DrawerTrigger : PopoverTrigger" as-child :disabled="disabled">
            <div tabindex="0" :class="cn(
                inputBaseClass,
                'flex items-center justify-between cursor-pointer py-2',
                disabled && 'pointer-events-none opacity-50',
                props.class
            )">
                <span :class="cn(
                    'truncate',
                    !displayValue && 'text-muted-foreground'
                )">
                    {{ displayValue || placeholder || 'Please select' }}
                </span>

                <div class="flex items-center gap-2">
                    <X v-if="allowClear && displayValue" class="h-4 w-4 shrink-0 opacity-50 hover:opacity-100"
                        @click.stop="clear" />
                    <ChevronDown class="h-4 w-4 shrink-0 opacity-50 hover:opacity-100" />
                </div>
            </div>
        </component>

        <!-- Mobile: Single level display -->
        <DrawerContent v-if="isMobile" class="p-0">
            <DrawerHeader>
                <DrawerTitle class="flex items-center gap-2 justify-between">
                    <span>{{ mobileCurrentTitle }}</span>
                    <button v-if="mobileActiveLevel > 0" @click="mobileActiveLevel--"
                        class="text-muted-foreground hover:text-foreground transition-colors" type="button">
                        <ArrowLeft class="w-4 h-4" />
                    </button>
                </DrawerTitle>
            </DrawerHeader>

            <div class="max-h-[70vh] overflow-auto">
                <div v-for="opt in mobileCurrentColumn" :key="opt.value"
                    class="flex justify-between items-center px-4 py-3 text-sm cursor-pointer hover:bg-accent transition-colors"
                    :class="[
                        opt.disabled && 'opacity-50 cursor-not-allowed pointer-events-none'
                    ]" @click="handleMobileSelect(opt)">
                    <span>{{ getLabel(opt) }}</span>
                    <ChevronRight v-if="opt.children?.length" class="w-4 h-4 opacity-50 shrink-0" />
                </div>
            </div>
        </DrawerContent>

        <!-- Desktop: Multi-column display -->
        <PopoverContent v-if="!isMobile" class="p-0 w-auto max-w-none" align="start">
            <div class="inline-flex">
                <div v-for="(column, colIndex) in columns" :key="colIndex"
                    class="min-w-[140px] max-h-[300px] overflow-auto" :class="{
                        'border-r': colIndex !== columns.length - 1
                    }">
                    <div v-for="opt in column" :key="opt.value"
                        class="flex justify-between px-3 py-1.5 cursor-pointer text-sm" :class="[
                            selectedValue[colIndex] === opt.value && 'bg-accent',
                            opt.disabled && 'opacity-50 cursor-not-allowed'
                        ]" @click="selectOption(opt, colIndex)"
                        @mouseenter="expandTrigger === 'hover' && opt.children && selectOption(opt, colIndex)">
                        <span>{{ getLabel(opt) }}</span>
                        <ChevronRight v-if="opt.children?.length" class="w-4 h-4 opacity-50" />
                    </div>
                </div>
            </div>
        </PopoverContent>
    </component>
</template>