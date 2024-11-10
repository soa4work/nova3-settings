<script>
export default {
  props: {
    title: String,
    description: String, 
    fields: Array,
    errors: Object
  }
}
</script>
<template>
  <div class="panel">
    <div class="panel-title">
      <h3 class="text-base font-bold">{{ title }}</h3>
      <p class="text-sm text-80">{{ description }}</p>
    </div>
    
    <card class="panel-content">
      <component
        v-for="(field, index) in fields"
        :key="index"
        :is="`form-${field.component}`"
        :field="field"
        :errors="errors"
      />
    </card>
  </div>
</template>
