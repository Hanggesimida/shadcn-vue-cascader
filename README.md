# Cascader-Shadcn-Vue

A cascading dropdown component for selecting hierarchical data such as locations, categories, or organizational structures.

Inspired by: Cascader components from Ant Design and React Suite

## Features

- Hierarchical cascading menu
- Click or hover expansion
- Supports icons and custom labels
- Custom display rendering
- Disable per option or entire component
- Responsive design with drawer mode on mobile
- Shadcn-compatible + Tailwind-friendly
- Fully type-safe (TypeScript)

## Installation

### Using Shadcn CLI

```bash
npx shadcn@latest add https://cascader-shadcn.surge.sh/r/cascader.json
```

### Manual Installation

Copy the `Cascader.vue` component from the repository into your Shadcn components directory (`src/components/ui/cascader/`).

## Usage

### Example.vue

```vue
<script setup lang="ts">
import { ref } from 'vue'
import Cascader from '@/components/ui/cascader/Cascader.vue'
import type { CascaderOption } from '@/components/ui/cascader/Cascader.vue'

const options: CascaderOption[] = [
  {
    value: "usa",
    label: "USA",
    children: [
      {
        value: "new_york",
        label: "New York",
        children: [
          { value: "statue_of_liberty", label: "Statue of Liberty" },
        ],
      },
    ],
  },
]

const selectedValue = ref<string[]>([])

function handleChange(value: string[], selectedOptions: CascaderOption[]) {
  console.log(value, selectedOptions)
}
</script>

<template>
  <Cascader
    v-model="selectedValue"
    :options="options"
    placeholder="Select location"
    @change="handleChange"
  />
</template>
```

## API Reference

### CascaderOption

| Property | Type | Description |
|----------|------|-------------|
| `value` | `string` | Unique option identifier |
| `label` | `string \| any` | Display label (text or component) |
| `textLabel` | `string` | Text label for display; fallback to value |
| `disabled` | `boolean` | Whether this option is disabled |
| `children` | `CascaderOption[]` | Nested options |

### Cascader Props

| Prop | Type | Default | Description |
|------|------|---------|-------------|
| `options` | `CascaderOption[]` | — | Cascader data options (required) |
| `modelValue` | `string[]` | — | Controlled selected value |
| `placeholder` | `string` | `"Please select"` | Placeholder text |
| `disabled` | `boolean` | `false` | Disable the component |
| `allowClear` | `boolean` | `false` | Show clear button |
| `expandTrigger` | `"click" \| "hover"` | `"click"` | How nested options expand |
| `class` | `string` | — | Trigger className |

### Cascader Events

| Event | Payload | Description |
|-------|---------|-------------|
| `update:modelValue` | `(value: string[]) => void` | Emitted when the value changes (for v-model) |
| `change` | `(value: string[], options: CascaderOption[]) => void` | Triggered when selection changes |

## Author

Built With 🍪 by Weihang Li

## License

MIT
