<template>
<div>
  <IndexComponent />
  <filters />
  
</div>
</template>

<script>
import Filters from '../components/Filters.vue'
export default {
  components: { Filters },
  name: 'SearchPage'
  
}
</script>
