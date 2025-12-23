<script setup lang="ts">
import { ref } from 'vue'
import Cascader from '@/components/ui/cascader/Cascader.vue'
import type { CascaderOption } from '@/components/ui/cascader/Cascader.vue'

// Example data: Region selection
const regionOptions: CascaderOption[] = [
  {
    value: 'china',
    label: 'China',
    children: [
      {
        value: 'beijing',
        label: 'Beijing',
        children: [
          { value: 'dongcheng', label: 'Dongcheng District' },
          { value: 'xicheng', label: 'Xicheng District' },
          { value: 'chaoyang', label: 'Chaoyang District' },
          { value: 'haidian', label: 'Haidian District' },
        ],
      },
      {
        value: 'shanghai',
        label: 'Shanghai',
        children: [
          { value: 'huangpu', label: 'Huangpu District' },
          { value: 'xuhui', label: 'Xuhui District' },
          { value: 'changning', label: 'Changning District' },
        ],
      },
      {
        value: 'guangdong',
        label: 'Guangdong',
        children: [
          { value: 'guangzhou', label: 'Guangzhou' },
          { value: 'shenzhen', label: 'Shenzhen' },
          { value: 'zhuhai', label: 'Zhuhai' },
        ],
      },
    ],
  },
  {
    value: 'usa',
    label: 'United States',
    children: [
      {
        value: 'california',
        label: 'California',
        children: [
          { value: 'losangeles', label: 'Los Angeles' },
          { value: 'sanfrancisco', label: 'San Francisco' },
        ],
      },
      {
        value: 'newyork',
        label: 'New York',
        children: [
          { value: 'manhattan', label: 'Manhattan' },
          { value: 'brooklyn', label: 'Brooklyn' },
        ],
      },
    ],
  },
]

// Example data: Product categories
const categoryOptions: CascaderOption[] = [
  {
    value: 'electronics',
    label: 'Electronics',
    children: [
      {
        value: 'phones',
        label: 'Phones',
        children: [
          { value: 'iphone', label: 'iPhone' },
          { value: 'android', label: 'Android' },
        ],
      },
      {
        value: 'computers',
        label: 'Computers',
        children: [
          { value: 'laptops', label: 'Laptops' },
          { value: 'desktops', label: 'Desktops' },
        ],
      },
    ],
  },
  {
    value: 'clothing',
    label: 'Clothing',
    children: [
      {
        value: 'mens',
        label: "Men's",
        children: [
          { value: 'shirts', label: 'Shirts' },
          { value: 'pants', label: 'Pants' },
        ],
      },
      {
        value: 'womens',
        label: "Women's",
        children: [
          { value: 'dresses', label: 'Dresses' },
          { value: 'shoes', label: 'Shoes' },
        ],
      },
    ],
  },
]

// Reactive data
const regionValue = ref<string[]>([])
const hoverValue = ref<string[]>([])
const disabledValue = ref<string[]>([])
const defaultValue = ref<string[]>(['china', 'beijing', 'chaoyang'])

// Handle change events
function handleRegionChange(value: string[], options: CascaderOption[]) {
  console.log('Region selected:', value, options)
}

function handleCategoryChange(value: string[], options: CascaderOption[]) {
  console.log('Category selected:', value, options)
}
</script>

<template>
  <div class="min-h-screen bg-background p-8">
    <div class="max-w-4xl mx-auto space-y-8">
      <!-- Title -->
      <div class="text-center space-y-2">
        <h1 class="text-4xl font-bold">shadcn-vue-cascader</h1>
        <p class="text-muted-foreground">A powerful cascader component</p>
      </div>

      <!-- Example cards -->
      <div class="grid gap-6">
        <!-- Basic example -->
        <div class="space-y-3 p-6 border rounded-lg bg-card">
          <div>
            <h2 class="text-xl font-semibold mb-1">Basic Usage</h2>
            <p class="text-sm text-muted-foreground">Click to select a region</p>
          </div>
          <Cascader
            v-model="regionValue"
            :options="regionOptions"
            placeholder="Please select a region"
            allow-clear
            @change="handleRegionChange"
          />
          <div v-if="regionValue.length" class="text-sm text-muted-foreground">
            Selected: {{ regionValue.join(' / ') }}
          </div>
        </div>

        <!-- Hover expand example -->
        <div class="space-y-3 p-6 border rounded-lg bg-card">
          <div>
            <h2 class="text-xl font-semibold mb-1">Hover Expand</h2>
            <p class="text-sm text-muted-foreground">Hover to expand sub-options</p>
          </div>
          <Cascader
            v-model="hoverValue"
            :options="categoryOptions"
            placeholder="Please select a category"
            expand-trigger="hover"
            allow-clear
            @change="handleCategoryChange"
          />
          <div v-if="hoverValue.length" class="text-sm text-muted-foreground">
            Selected: {{ hoverValue.join(' / ') }}
          </div>
        </div>

        <!-- Disabled state example -->
        <div class="space-y-3 p-6 border rounded-lg bg-card">
          <div>
            <h2 class="text-xl font-semibold mb-1">Disabled State</h2>
            <p class="text-sm text-muted-foreground">Component is disabled</p>
          </div>
          <Cascader
            v-model="disabledValue"
            :options="regionOptions"
            placeholder="Please select a region"
            disabled
          />
        </div>

        <!-- Default value example -->
        <div class="space-y-3 p-6 border rounded-lg bg-card">
          <div>
            <h2 class="text-xl font-semibold mb-1">Default Value</h2>
            <p class="text-sm text-muted-foreground">Component with preset default value</p>
          </div>
          <Cascader
            v-model="defaultValue"
            :options="regionOptions"
            placeholder="Please select a region"
            allow-clear
          />
        </div>
      </div>

      <!-- Features -->
      <div class="mt-8 p-6 border rounded-lg bg-card">
        <h2 class="text-xl font-semibold mb-4">Features</h2>
        <ul class="space-y-2 text-sm text-muted-foreground">
          <li class="flex items-start gap-2">
            <span class="text-primary">•</span>
            <span>Multi-level cascading selection</span>
          </li>
          <li class="flex items-start gap-2">
            <span class="text-primary">•</span>
            <span>Responsive design, automatically switches to drawer mode on mobile</span>
          </li>
          <li class="flex items-start gap-2">
            <span class="text-primary">•</span>
            <span>Supports both click and hover expand modes</span>
          </li>
          <li class="flex items-start gap-2">
            <span class="text-primary">•</span>
            <span>Clear selection support</span>
          </li>
          <li class="flex items-start gap-2">
            <span class="text-primary">•</span>
            <span>Disabled state support</span>
          </li>
          <li class="flex items-start gap-2">
            <span class="text-primary">•</span>
            <span>Fully type-safe (TypeScript)</span>
          </li>
        </ul>
      </div>
    </div>
  </div>
</template>

<style scoped>
</style>