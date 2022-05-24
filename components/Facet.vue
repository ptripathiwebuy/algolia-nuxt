<template>
  <ais-refinement-list :limit="400" :attribute="attributeName" >
    <template
      v-slot="{
      items,
      isShowingMore,
      isFromSearch,
      canToggleShowMore,
      refine,
      createURL,
      toggleShowMore,      
      sendEvent,
    }"
    >
       <div class="refine-cat">       
        <p class="refine-by-text">{{title}}</p>
        <template v-if="!isFromSearch && items.length">
        <div class="refine-select-div color-search-div" v-if="items.length >= 8">
          <div class="custom-input-div">
            <input
              type="text"
              @input="searchForItems($event.currentTarget.value, items)"
              
            />
          </div>
        </div>
        <ul>
          
          <template v-for="(item,key) in items" v-if="key < 8 || showmore">
            <li :key="item.value" :class="isHorizontal ? 'horizontal' : ''">
              <label
                :class="{'boldtext': item.isRefined, 'attributetxt': isHorizontal}"
                class="refine-checkbox"
              >
                <input
                  :disabled="item.disabled"
                  type="checkbox"
                  v-model="item.isRefined"
                  @change="refine(item.value);scrollToTop()"
                  :value="item.value"
                />
                <p>
                  <span :class="{'refine-graytxt': item.disabled}">{{ item.value }}</span>
                </p>
                <span v-if="!isHorizontal">({{ item.count }})</span>
              </label>
            </li>
          </template>
        </ul>
        <a
          href="javascript:void(0)"
          @click.prevent="showmore = true"
          class="show-more-txt"
          v-if="filteredItems.length > 8 && !showmore"
          >show more</a
        >
        <a
          href="javascript:void(0)"
          @click.prevent="showmore = false"
          class="show-more-txt"
          v-if="filteredItems.length > 8 && showmore"
          >show less</a
        >
        </template>      
        <div v-else style="color:red">dfdfd</div>
      </div>
    </template>
  </ais-refinement-list>
</template>

<script>

const FACET_OR = 'or'
const FACET_AND = 'and'

export default {
    props: {
        attributeName: {
            type: String,
            required: true
        },
        operator: {
            type: String,
            default: FACET_OR,
            validator (rawValue) {
                const value = rawValue.toLowerCase()

                return value === FACET_OR || value === FACET_AND
            }
        },
        limit: {
            type: Number,
            default: 400
        },
        sortBy: {
            default () {
                return ['isRefined:desc', 'count:desc', 'name:asc']
            }
        },
        title: {},
        isShowMore: {
            type: Boolean,
            default: true
        },
        isHorizontal: {
            type: Boolean,
            default: false
        }
    },
    data () {
        return {
            blockClassName: 'ais-refinement-list',
            searchText: '',
            toshowmore: true,
            storeList: [],
            filteredItems: [],
            showmore: false
        }
    },
    methods: {
      getAttributeValue(items, value, column, defaultValue) {
        const selectedValue = items.find(item => item.value == value)               
        if (selectedValue) {          
          return selectedValue[column]         
        }        
        return defaultValue
      },
     
      scrollToTop() {
        window.scrollTo({
                top: 0,
                behavior: 'smooth'
            })
      },
      searchForItems(search, Items) {       
        this.filteredItems =  Items.filter(facet => facet.value.toLowerCase().includes(search.toLowerCase()))
      }
    }
}
</script>
