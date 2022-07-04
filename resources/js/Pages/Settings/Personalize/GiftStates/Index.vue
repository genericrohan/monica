<style lang="scss" scoped>
.item-list {
  &:hover:first-child {
    border-top-left-radius: 8px;
    border-top-right-radius: 8px;
  }

  &:hover:last-child {
    border-bottom-left-radius: 8px;
    border-bottom-right-radius: 8px;
  }
}
</style>

<template>
  <layout :layout-data="layoutData">
    <!-- breadcrumb -->
    <nav class="bg-white sm:border-b">
      <div class="max-w-8xl mx-auto hidden px-4 py-2 sm:px-6 md:block">
        <div class="flex items-baseline justify-between space-x-6">
          <ul class="text-sm">
            <li class="mr-2 inline text-gray-600">{{ $t('app.breadcrumb_location') }}</li>
            <li class="mr-2 inline">
              <inertia-link :href="data.url.settings" class="text-blue-500 hover:underline">{{
                $t('app.breadcrumb_settings')
              }}</inertia-link>
            </li>
            <li class="relative mr-2 inline">
              <svg
                xmlns="http://www.w3.org/2000/svg"
                class="icon-breadcrumb relative inline h-3 w-3"
                fill="none"
                viewBox="0 0 24 24"
                stroke="currentColor">
                <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M9 5l7 7-7 7" />
              </svg>
            </li>
            <li class="mr-2 inline">
              <inertia-link :href="data.url.personalize" class="text-blue-500 hover:underline"
                >Personalize your account</inertia-link
              >
            </li>
            <li class="relative mr-2 inline">
              <svg
                xmlns="http://www.w3.org/2000/svg"
                class="icon-breadcrumb relative inline h-3 w-3"
                fill="none"
                viewBox="0 0 24 24"
                stroke="currentColor">
                <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M9 5l7 7-7 7" />
              </svg>
            </li>
            <li class="inline">Gift states</li>
          </ul>
        </div>
      </div>
    </nav>

    <main class="relative sm:mt-20">
      <div class="mx-auto max-w-3xl px-2 py-2 sm:py-6 sm:px-6 lg:px-8">
        <!-- title + cta -->
        <div class="mb-6 mt-8 items-center justify-between sm:mt-0 sm:flex">
          <h3 class="mb-4 sm:mb-0"><span class="mr-1"> 🎁 </span> All the gift states</h3>
          <pretty-button
            v-if="!createGiftStateModalShown"
            :text="'Add a gift state'"
            :icon="'plus'"
            @click="showGiftStateModal" />
        </div>

        <!-- modal to create a gift state -->
        <form
          v-if="createGiftStateModalShown"
          class="mb-6 rounded-lg border border-gray-200 bg-white"
          @submit.prevent="submit()">
          <div class="border-b border-gray-200 p-5">
            <errors :errors="form.errors" />

            <text-input
              :ref="'newGiftState'"
              v-model="form.label"
              :label="'Name'"
              :type="'text'"
              :autofocus="true"
              :input-class="'block w-full'"
              :required="true"
              :autocomplete="false"
              :maxlength="255"
              @esc-key-pressed="createGiftStateModalShown = false" />
          </div>

          <div class="flex justify-between p-5">
            <pretty-span :text="$t('app.cancel')" :classes="'mr-3'" @click="createGiftStateModalShown = false" />
            <pretty-button :text="$t('app.save')" :state="loadingState" :icon="'plus'" :classes="'save'" />
          </div>
        </form>

        <!-- list of gift states -->
        <div v-if="localGiftStates.length > 0" class="mb-6">
          <draggable
            :list="localGiftStates"
            item-key="id"
            :component-data="{ name: 'fade' }"
            handle=".handle"
            @change="updatePosition">
            <template #item="{ element }">
              <div v-if="editGiftStateId != element.id" class="">
                <div
                  class="item-list mb-2 flex items-center justify-between rounded-lg border border-gray-200 bg-white py-2 pl-4 pr-5 hover:bg-slate-50">
                  <!-- icon to move position -->
                  <div class="mr-2 flex">
                    <svg
                      class="handle mr-2 cursor-move"
                      width="24"
                      height="24"
                      viewBox="0 0 24 24"
                      fill="none"
                      xmlns="http://www.w3.org/2000/svg">
                      <path d="M7 7H9V9H7V7Z" fill="currentColor" />
                      <path d="M11 7H13V9H11V7Z" fill="currentColor" />
                      <path d="M17 7H15V9H17V7Z" fill="currentColor" />
                      <path d="M7 11H9V13H7V11Z" fill="currentColor" />
                      <path d="M13 11H11V13H13V11Z" fill="currentColor" />
                      <path d="M15 11H17V13H15V11Z" fill="currentColor" />
                      <path d="M9 15H7V17H9V15Z" fill="currentColor" />
                      <path d="M11 15H13V17H11V15Z" fill="currentColor" />
                      <path d="M17 15H15V17H17V15Z" fill="currentColor" />
                    </svg>

                    <span>{{ element.label }}</span>
                  </div>

                  <!-- actions -->
                  <ul class="text-sm">
                    <li
                      class="inline cursor-pointer text-blue-500 hover:underline"
                      @click="renameGiftStateModal(element)">
                      Rename
                    </li>
                    <li class="ml-4 inline cursor-pointer text-red-500 hover:text-red-900" @click="destroy(element)">
                      Delete
                    </li>
                  </ul>
                </div>

                <div class="ml-3 mb-3">👇</div>
              </div>

              <form
                v-else
                class="item-list border-b border-gray-200 hover:bg-slate-50"
                @submit.prevent="update(element)">
                <div class="border-b border-gray-200 p-5">
                  <errors :errors="form.errors" />

                  <text-input
                    :ref="'rename' + element.id"
                    v-model="form.label"
                    :label="'Name'"
                    :type="'text'"
                    :autofocus="true"
                    :input-class="'block w-full'"
                    :required="true"
                    :autocomplete="false"
                    :maxlength="255"
                    @esc-key-pressed="editGiftStateId = 0" />
                </div>

                <div class="flex justify-between p-5">
                  <pretty-span :text="$t('app.cancel')" :classes="'mr-3'" @click.prevent="editGiftStateId = 0" />
                  <pretty-button :text="'Rename'" :state="loadingState" :icon="'check'" :classes="'save'" />
                </div>
              </form>
            </template>
          </draggable>
        </div>

        <!-- blank state -->
        <div v-if="localGiftStates.length == 0" class="mb-6 rounded-lg border border-gray-200 bg-white">
          <p class="p-5 text-center">Gift states let you define the various states for your gifts.</p>
        </div>
      </div>
    </main>
  </layout>
</template>

<script>
import Layout from '@/Shared/Layout';
import PrettyButton from '@/Shared/Form/PrettyButton';
import PrettySpan from '@/Shared/Form/PrettySpan';
import TextInput from '@/Shared/Form/TextInput';
import Errors from '@/Shared/Form/Errors';
import draggable from 'vuedraggable';

export default {
  components: {
    Layout,
    PrettyButton,
    PrettySpan,
    TextInput,
    Errors,
    draggable,
  },

  props: {
    layoutData: {
      type: Object,
      default: null,
    },
    data: {
      type: Object,
      default: null,
    },
  },

  data() {
    return {
      loadingState: '',
      createGiftStateModalShown: false,
      editGiftStateId: 0,
      localGiftStates: [],
      form: {
        label: '',
        position: '',
        errors: [],
      },
    };
  },

  mounted() {
    this.localGiftStates = this.data.gift_states;
  },

  methods: {
    showGiftStateModal() {
      this.form.label = '';
      this.form.position = '';
      this.createGiftStateModalShown = true;

      this.$nextTick(() => {
        this.$refs.newGiftState.focus();
      });
    },

    renameGiftStateModal(giftState) {
      this.form.label = giftState.label;
      this.editGiftStateId = giftState.id;
    },

    submit() {
      this.loadingState = 'loading';

      axios
        .post(this.data.url.store, this.form)
        .then((response) => {
          this.flash('The gift state has been created', 'success');
          this.localGiftStates.push(response.data.data);
          this.loadingState = null;
          this.createGiftStateModalShown = false;
        })
        .catch((error) => {
          this.loadingState = null;
          this.form.errors = error.response.data;
        });
    },

    update(giftState) {
      this.loadingState = 'loading';

      axios
        .put(giftState.url.update, this.form)
        .then((response) => {
          this.flash('The gift state has been updated', 'success');
          this.localGiftStates[this.localGiftStates.findIndex((x) => x.id === giftState.id)] = response.data.data;
          this.loadingState = null;
          this.editGiftStateId = 0;
        })
        .catch((error) => {
          this.loadingState = null;
          this.form.errors = error.response.data;
        });
    },

    destroy(giftState) {
      if (confirm('Are you sure? This can not be undone.')) {
        axios
          .delete(giftState.url.destroy)
          .then((response) => {
            this.flash('The gift state has been deleted', 'success');
            var id = this.localGiftStates.findIndex((x) => x.id === giftState.id);
            this.localGiftStates.splice(id, 1);
          })
          .catch((error) => {
            this.loadingState = null;
            this.form.errors = error.response.data;
          });
      }
    },

    updatePosition(event) {
      // the event object comes from the draggable component
      this.form.position = event.moved.newIndex + 1;

      axios
        .post(event.moved.element.url.position, this.form)
        .then((response) => {
          this.flash('The order has been saved', 'success');
        })
        .catch((error) => {
          this.loadingState = null;
          this.errors = error.response.data;
        });
    },
  },
};
</script>