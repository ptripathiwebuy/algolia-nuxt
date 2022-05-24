<template>
   <ais-state-results>
      <template v-slot="{state,results}">
         <div class="prod-attribute-filter-div" :class="{'mh50': hasResultforQuery(state, results)}">
            <div class="refine-txt-div" v-if="hasResultforQuery(state, results)">
               <p class="refine-txt">Refine  </p>
               <ais-clear-refinements>
                  <template v-slot="{ refine, createURL }">      
                     <a :href="createURL()" @click.prevent="refine" class="resetbuttonfilter" title="Reset">Reset</a>
                  </template>
               </ais-clear-refinements>
            </div>
            <Facet attribute-name="availability" title="availability" />
            <Facet attribute-name="stores" title="stores" />
            <Facet attribute-name="categoryFriendlyName" title="categoryFriendlyName" v-show="hasResultforQuery(state, results)" />           
            <template v-if="results">
               <template v-for="attribute in getAttributes(results)">
                  <Facet :key="attribute" :attribute-name="attribute" :title="attribute"  :isHorizontal="true"  />
               </template>
            </template>
         </div>
      </template>
   </ais-state-results>
</template>
<script>

import Facet from '~~/components/Facet.vue'
// import PriceFacet from '~~/components/desktop/boxsearch/PriceFilter.vue'

export default {
    name: 'Filters',    
    components: {Facet},    
    methods: {
        hasResultforQuery(state,results) {
            return results.hits.length > 0            
        },        
        getAttributes(results) {
            const rawResult = results?._rawResults[0]?.facets            
            const allFilters = rawResult ?  Object.keys(rawResult)  : []
            const staticFilters = ['sellPrice', 'availability', 'stores', 'categoryFriendlyName']
            const dynamicFilters = allFilters.filter(allFilter => !staticFilters.includes(allFilter))            
            return dynamicFilters          
        }
    }
}
</script>

<style>
.mh50 {
    min-height: 50px;
}
</style>