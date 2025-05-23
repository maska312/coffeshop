<template>
  <div class="bg-white">
    <div class="mx-auto max-w-2xl px-4 py-16 sm:px-6 sm:py-24 lg:grid lg:max-w-7xl lg:grid-cols-12 lg:gap-x-8 lg:px-8 lg:py-32">
      <div class="lg:col-span-4">
        <h2 class="text-2xl font-bold tracking-tight text-gray-900">Customer Reviews</h2>

        <div class="mt-3 flex items-center">
          <div>
            <div class="flex items-center">
              <StarIcon v-for="rating in [0, 1, 2, 3, 4]" :key="rating" :class="[averageRating > rating ? 'text-yellow-400' : 'text-gray-300', 'h-5 w-5 flex-shrink-0']" aria-hidden="true" />
            </div>
            <p class="sr-only">{{ averageRating }} out of 5 stars</p>
          </div>
          <p class="ml-2 text-sm text-gray-900">Based on {{ reviews.length }} reviews</p>
        </div>

        <div class="mt-6">
          <h3 class="sr-only">Review data</h3>

          <dl class="space-y-3">
            <div v-for="count in reviews.counts" :key="count.rating" class="flex items-center text-sm">
              <dt class="flex flex-1 items-center">
                <p class="w-3 font-medium text-gray-900">{{ count.rating }}<span class="sr-only"> star reviews</span></p>
                <div aria-hidden="true" class="ml-1 flex flex-1 items-center">
                  <StarIcon :class="[count.count > 0 ? 'text-yellow-400' : 'text-gray-300', 'h-5 w-5 flex-shrink-0']" aria-hidden="true" />

                  <div class="relative ml-3 flex-1">
                    <div class="h-3 rounded-full border border-gray-200 bg-gray-100" />
                    <div v-if="count.count > 0" class="absolute inset-y-0 rounded-full border border-yellow-400 bg-yellow-400" :style="{ width: `calc(${count.count} / ${reviews.totalCount} * 100%)` }" />
                  </div>
                </div>
              </dt>
              <dd class="ml-3 w-10 text-right text-sm tabular-nums text-gray-900">{{ Math.round((count.count / reviews.totalCount) * 100) }}%</dd>
            </div>
          </dl>
        </div>

        <div class="mt-10">
          <h3 class="text-lg font-medium text-gray-900">Share your thoughts</h3>
          <p class="mt-1 text-sm text-gray-600">If you’ve used this product, share your thoughts with other customers</p>

          <button v-if="isSignedIn" @click="toggleReviewForm" class="mt-6 inline-flex w-full items-center justify-center rounded-md border border-gray-300 bg-white px-8 py-2 text-sm font-medium text-gray-900 hover:bg-gray-50 sm:w-auto lg:w-full">
            {{ userHasReview ? 'Edit your review' : 'Write a review' }}
          </button>
          <router-link to="/signin" v-else class="mt-6 inline-flex w-full items-center justify-center rounded-md border border-gray-300 bg-white px-8 py-2 text-sm font-medium text-gray-900 sm:w-auto lg:w-full">
            Please sign in to write a review
          </router-link>
        </div>
      </div>

      <div class="mt-16 lg:col-span-7 lg:col-start-6 lg:mt-0">
        <h3 class="sr-only">Recent reviews</h3>

        <div v-if="reviews.length" class="flow-root">
          <div class="-my-12 divide-y divide-gray-200">
            <div v-for="review in reviews" :key="review.id" class="py-12">
              <div class="flex items-center relative">
                <div class="flex flex-col gap-y-1">
                  <h4 class="text-sm font-bold text-gray-900">{{ review.author }}</h4>
                  <div class="mt-1 flex items-center">
                    <StarIcon v-for="rating in [0, 1, 2, 3, 4]" :key="rating" :class="[review.rating > rating ? 'text-yellow-400' : 'text-gray-300', 'h-5 w-5 flex-shrink-0']" aria-hidden="true" />
                    <span class="text-xs ml-2 text-green-600 font-semibold" v-if="review.verifiedPurchase" >Verified Purchase</span>
                  </div>
                  <span class="text-xs text-gray-400" v-if="review.timestamp" >{{ formatDate(review.timestamp) }} <span v-if="review.edited" >( Edited )</span></span>
                  <p class="sr-only">{{ review.rating }} out of 5 stars</p>
                </div>
              </div>

              <div class="mt-4 space-y-6 text-base italic text-gray-600" v-html="review.content" />
            </div>
          </div>
        </div>
        <p v-else class="text-gray-500 border-b pb-5">
          No Reviews.
        </p>
      </div>
    </div>

    <ReviewFormModal v-if="showReviewForm" @close="toggleReviewForm" :productId="productId"/>
  </div>
</template>

<script setup>
import { ref, computed, onMounted } from 'vue';
import { useStore } from 'vuex';
import { useRoute } from 'vue-router';
import { StarIcon } from '@heroicons/vue/20/solid';
import ReviewFormModal from '@app-components/ReviewFormModal.vue';

const store = useStore();
const route = useRoute();

const showReviewForm = ref(false);

const isSignedIn = computed(() => store.getters['auth/isSignedIn']);
const userId = computed(() => store.getters['auth/user']?.id);
const productId = computed(() => parseInt(route.params.id));

const reviews = computed(() => store.getters['reviews/reviewsByProductSorted'](productId.value));
const averageRating = computed(() => store.getters['reviews/averageRatingByProduct'](productId.value));
const userReview = computed(() => {
  return store.getters['reviews/reviewsByUser'](userId.value).find(review => review.productId === productId.value);
});
const userHasReview = computed(() => !!userReview.value);
const toggleReviewForm = () => {
  showReviewForm.value = !showReviewForm.value;
};

const formatDate = (timestamp) => {
  const date = new Date(timestamp);
  return date.toLocaleDateString('en-GB', { year: 'numeric', month: 'long', day: 'numeric' });
};

onMounted(() => {
  if (productId.value) {
    store.dispatch('reviews/fetchReviewsByProduct', productId.value);
  }
  if (userId.value) {
    store.dispatch('reviews/fetchReviewsByUser', userId.value);
  }
});
</script>
