<template>
  <nav class="sf-pagination">
    <!-- @slot Custom markup for previous page button -->
    <slot name="prev" v-bind="{ isDisabled: !canGoPrev, go, prev: getPrev }">
      <div v-if="hasArrows" class="sf-pagination__item prev">
        <component
          :is="componentIs"
          v-if="canGoPrev"
          :class="{
            'sf-button--pure': !hasRouter,
          }"
          :link="hasRouter ? getLinkTo(getPrev) : null"
          aria-label="Go to previous page"
          @click="hasRouter ? null : go(getPrev)"
        >
          <SfIcon icon="arrow_left" size="1.125rem" />
        </component>
      </div>
    </slot>
    <template v-if="showFirst">
      <slot name="number" v-bind="{ page: 1 }">
        <component
          :is="componentIs"
          class="sf-pagination__item"
          :class="{
            'sf-button--pure': !hasRouter,
          }"
          :link="hasRouter ? getLinkTo(1) : null"
          @click="hasRouter ? null : go(1)"
        >
          1
        </component>
      </slot>
      <slot v-if="firstVisiblePageNumber > 2" name="points">
        <div class="sf-pagination__item">...</div>
      </slot>
    </template>
    <template v-for="page in limitedPageNumbers">
      <slot name="number" v-bind="{ page, currentPage }">
        <component
          :is="currentPage === page ? 'span' : componentIs"
          :key="page"
          class="sf-pagination__item"
          :class="{
            'sf-button--pure': !hasRouter && currentPage !== page,
            current: currentPage === page,
          }"
          :link="hasRouter && currentPage !== page ? getLinkTo(page) : null"
          @click="!hasRouter && currentPage !== page ? go(page) : null"
        >
          {{ page }}
        </component>
      </slot>
    </template>
    <template v-if="showLast">
      <slot v-if="lastVisiblePageNumber < total - 1" name="points">
        <div class="sf-pagination__item">...</div>
      </slot>
      <slot name="number" v-bind="{ page: total }">
        <component
          :is="componentIs"
          class="sf-pagination__item"
          :class="{
            'sf-button--pure': !hasRouter,
          }"
          :link="hasRouter ? getLinkTo(total) : null"
          @click="hasRouter ? null : go(total)"
        >
          {{ total }}
        </component>
      </slot>
    </template>
    <!-- @slot Custom markup for previous page button -->
    <slot name="next" v-bind="{ isDisabled: !canGoNext, go, next: getNext }">
      <div v-if="hasArrows" class="sf-pagination__item next">
        <component
          :is="componentIs"
          v-if="canGoNext"
          :class="{
            'sf-button--pure': !hasRouter,
          }"
          :link="hasRouter ? getLinkTo(getNext) : null"
          aria-label="Go to previous next"
          @click="hasRouter ? null : go(getNext)"
        >
          <SfIcon icon="arrow_right" size="1.125rem" />
        </component>
      </div>
    </slot>
  </nav>
</template>
<script>
import SfIcon from "../../atoms/SfIcon/SfIcon.vue";
import SfLink from "../../atoms/SfLink/SfLink.vue";
import SfButton from "../../atoms/SfButton/SfButton.vue";
export default {
  name: "SfPagination",
  components: {
    SfIcon,
    SfLink,
    SfButton,
  },
  props: {
    /**
     * Total number of pages
     */
    total: {
      type: Number,
      default: 0,
    },
    /**
     * Maximum visible pagination items
     */
    visible: {
      type: Number,
      default: 5,
    },
    /**
     * Status of arrows display
     */
    hasArrows: {
      type: Boolean,
      default: true,
    },
    /**
     * Current page number, for non router
     */
    current: {
      type: Number,
      default: 1,
    },
    /**
     * Name of page query param for router
     */
    pageParamName: {
      type: String,
      default: "page",
    },
  },
  computed: {
    hasRouter() {
      return this.$route;
    },
    componentIs() {
      return this.hasRouter ? "SfLink" : "SfButton";
    },
    currentPage() {
      return this.hasRouter
        ? this.$route.query[this.pageParamName]
          ? parseInt(this.$route.query[this.pageParamName], 10)
          : 1
        : this.current;
    },
    getPrev() {
      return this.currentPage - 1;
    },
    canGoPrev() {
      return this.getPrev > 0;
    },
    getNext() {
      return this.currentPage + 1;
    },
    canGoNext() {
      return this.getNext <= this.total;
    },
    showFirst() {
      return this.firstVisiblePageNumber > 1;
    },
    showLast() {
      return this.lastVisiblePageNumber < this.total;
    },
    listOfPageNumbers() {
      return Array.from(Array(this.total), (_, i) => i + 1);
    },
    limitedPageNumbers() {
      if (this.total <= this.visible) {
        return this.listOfPageNumbers;
      }
      if (this.currentPage < this.visible - Math.floor(this.visible / 2) + 1) {
        return this.listOfPageNumbers.slice(0, this.visible);
      }
      if (
        this.total - this.currentPage <
        this.visible - Math.ceil(this.visible / 2) + 1
      ) {
        return this.listOfPageNumbers.slice(this.total - this.visible);
      }
      return this.listOfPageNumbers.slice(
        this.currentPage - Math.ceil(this.visible / 2),
        this.currentPage + Math.floor(this.visible / 2)
      );
    },
    firstVisiblePageNumber() {
      return this.limitedPageNumbers[0];
    },
    lastVisiblePageNumber() {
      return this.limitedPageNumbers[this.limitedPageNumbers.length - 1];
    },
  },
  methods: {
    go(page) {
      this.$emit("click", page);
    },
    getLinkTo(page) {
      const route = {
        name: this.$route.name,
        params: this.$route.params,
        query: { ...this.$route.query, page: page.toString() },
      };
      const linkToPage = this.$router.resolve(route).href;
      return linkToPage.slice(1);
    },
  },
};
</script>
<style lang="scss">
@import "~@storefront-ui/shared/styles/components/molecules/SfPagination.scss";
</style>
