<template>
  <ais-instant-search :search-client="searchClient" index-name="dev_cex_uk" :routing="routing" :middlewares="middlewares">
      <ais-configure :hits-per-page.camel="hitsPerPage" :facets="['*']"  :filters="filters" :maxValuesPerFacet="400"/>
        
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
    const vueRouter = this.$router; /* get this from Vue Router */
    const indexName = 'dev_cex_uk';
        function buildRouter(vueRouter) {
          return {
            _onPopState() {
              return null;
            },
            read() {
              return vueRouter.currentRoute.query;
            },
            isSearchPage() {
              return vueRouter.currentRoute.name == 'search'
            },
            write(routeState) {
              // Only push a new entry if the URL changed (avoid duplicated entries in the history)
              if (this.createURL(routeState) === this.createURL(this.read())) {
                return;
              }
        
              vueRouter.push({
                query: routeState,
              });
            },
            createURL(routeState) {
              return vueRouter.resolve({
                query: routeState,
              }).href;
            },
            onUpdate(cb) {
              if (typeof window === 'undefined') return;
        
              // This ensures updates from the vue Router (i.e. outside Instantsearch,
              // like clicking on a link in the navbar) propagate back to instantsearch,
              // so the uiState can be synchronised accordingly.
              // See https://github.com/algolia/vue-instantsearch/issues/930#issuecomment-802685017
              // vueRouter.afterEach() returns an unregiter function.
              // see https://github.com/vuejs/vue-router/issues/1318#issuecomment-296378495
              // By running it before leaving the page, the hook won't be invoked anymore.
              // When re-visting a new hook will take its place instead of pushing a new,
              // identical one to the array of hooks, which would break the routing.
              // Notice that this unregister function is called within the dispose() method
              this.removeAfterEachHook = vueRouter.afterEach(() => {
                cb(this.read());
              });
        
              // Removed these lines
              // this._onPopState = (event) => {
              //   const routeState = event?.state;
              //   // On initial load, the state is read from the URL without
              //   // update. Therefore, the state object isn't present. In this
              //   // case, we fallback and read the URL.
              //   if (!routeState) {
              //     cb(this.read());
              //   } else {
              //     cb(routeState);
              //   }
              // };
              // window.addEventListener('popstate', this._onPopState);
            },
            dispose() {
              if (typeof window === 'undefined') return;
        
              // Removed this line
              // window.removeEventListener('popstate', this._onPopState);
              this.removeAfterEachHook();
            },
          };
        }

const routing = buildRouter(vueRouter)
let $this = this
const stateMapping  = {
          stateToRoute(uiState) {
                                   
            const indexUiState = uiState[indexName]            
            if (indexUiState.query && (!$this.$route.query.superCatId || $this.$route.query.superCatName)) {
               $this.newFilters = ''
            }                      
            const returnRoute =  {
              stext: indexUiState.query,                  
              categoryIds:  indexUiState.query ?  undefined : $this.$route.query.categoryIds,
              categoryName: indexUiState.query ?  undefined : $this.$route.query.categoryName,
              superCatId:   indexUiState.query && $this.$route.query.superCatName ?  undefined : $this.$route.query.superCatId,    
              superCatName: indexUiState.query ?  undefined : $this.$route.query.superCatName        
            };                 
            for (let i in indexUiState.refinementList) {
                returnRoute[i] = indexUiState.refinementList[i] && indexUiState.refinementList[i].join(',')
            }
            if (indexUiState.range  && indexUiState.range.sellPrice) {
               returnRoute.price =  indexUiState.range.sellPrice
            }
            if (indexUiState.sortBy && indexUiState.sortBy != indexName) {
              returnRoute.sortBy = indexUiState.sortBy
            }                       
            return returnRoute
          },
          routeToState(routeState) {                                                       
            const returnRouteState =  {
              [indexName]: {
                query: routeState.stext,                
                refinementList: {            
                },
                range: {
                  sellPrice: routeState.price
                },               
              },
            };
            for (let i in routeState) {
                if (i == 'stext' || i == 'price' || i == 'categoryIds' || i == 'categoryName' || i == 'superCatId' || i == 'superCatName' || i == 'sortBy') continue
                returnRouteState[indexName].refinementList[i] = routeState[i] && routeState[i].split(',')
            }
            if (routeState.sortBy && routeState.sortBy != indexName) {
              returnRouteState[indexName].sortBy = routeState.sortBy
            }                    
            return returnRouteState            
          },
        }
    return {
              middlewares: [insightsMiddleware],

      searchClient: algoliasearch(
        "testingKMXS1FTYRV",
        "ebccc210523a1bc0494eba243f3ac04b"
      ),
      routing: {
        router: historyRouter(),
        stateMapping: singleIndexMapping('dev_cex_uk'),
      },
      routing1: {
        router : routing,
        stateMapping: stateMapping
      },
    };
  },
};
</script>