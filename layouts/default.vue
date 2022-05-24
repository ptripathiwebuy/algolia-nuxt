<template>
  <ais-instant-search :search-client="searchClient" index-name="dev_cex_uk" :routing="routing" :middlewares="middlewares">
      <ais-configure :hits-per-page.camel="hitsPerPage" :facets="['*']" :filters="filters" :maxValuesPerFacet="400"/>
        
   <ais-search-box>
  <template v-slot="{ currentRefinement, isSearchStalled, refine }">
    <input
      type="search"
      :value="currentRefinement"
      @input="refine($event.currentTarget.value)"
      v-on:keyup.enter="$router.push({name:'search', query: {'query': currentRefinement}})"
    >
    <span :hidden="!isSearchStalled">Loading...</span>
  </template>
</ais-search-box>
   
    <nuxt />
  </ais-instant-search>
</template>

<script>
import algoliasearch from "algoliasearch/lite";
import { history as historyRouter } from 'instantsearch.js/es/lib/routers';
import { singleIndex as singleIndexMapping } from 'instantsearch.js/es/lib/stateMappings';
import { createInsightsMiddleware } from 'instantsearch.js/es/middlewares'
import aa from 'search-insights';


const insightsMiddleware = createInsightsMiddleware({
  insightsClient: aa,
})

aa('setUserToken', 'abcd')
export default {
    computed: {
        hitsPerPage() {
            
            return this.$route.name == 'search' ? 50 : 10
        },
        filters () {         
          let filter = ''
          if (this.$route.query.superCatId) {
              filter = filter + 'scId=' + this.$route.query.superCatId
          }
          if (this.$route.query.categoryIds) {
              filter = filter + 'categoryId=' + this.$route.query.categoryIds
          }                   
          return filter
      },
    },
  data() {    
    return {
              middlewares: [insightsMiddleware],

      searchClient: algoliasearch(
        "testingKMXS1FTYRV",
        "ebccc210523a1bc0494eba243f3ac04b"
      ),
      routing: {
        router: historyRouter(),
        stateMapping: singleIndexMapping('dev_cex_uk'),
      }      
    };
  },
};
</script>