<style lang="scss" scoped>
</style>

<template>
  <div>
    <div class="border-b mb-4 pb-3 sm:flex items-center justify-between sm:mt-0 mt-8">
      <h3>
        Pages
      </h3>
      <pretty-button v-if="!createPageModalShown" :text="'Add a page'" :icon="'plus'" @click="showPageModal" />
    </div>

    <!-- contact information page | can't be removed -->
    <div :class="isSelectedId == data.template_page_contact_information.id ? 'border-2	bg-sky-100' : ''" class="bg-white border hover:bg-slate-50 border-gray-200 rounded-lg mb-2 px-5 py-2 flex items-center" @click="selectPage(data.template_page_contact_information)">
      <!-- detail of a page -->
      <div>
        <div class="mb-0 block">
          {{ data.template_page_contact_information.name }}
        </div>

        <ul class="text-xs text-gray-400">
          <li class="inline">Can't be moved or deleted</li>
        </ul>
      </div>
    </div>

    <!-- list of pages that can be deleted -->
    <draggable
      :list="localPages"
      item-key="id"
      :component-data="{name:'fade'}"
      handle=".handle"
      @change="updatePosition"
    >
      <template #item="{ element }">
        <div v-if="renamePageModalShownId != element.id" :class="isSelectedId == element.id ? 'border-2	bg-sky-100' : ''" class="bg-white border hover:bg-slate-50 border-gray-200 rounded-lg mb-2 pl-2 pr-5 py-2 flex items-center" @click="selectPage(element)">
          <!-- icon to move position -->
          <div class="mr-2">
            <svg class="cursor-move handle" width="24" height="24" viewBox="0 0 24 24" fill="none"
                 xmlns="http://www.w3.org/2000/svg"
            >
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
          </div>

          <!-- detail of a page -->
          <div>
            <div class="mb-0 block">
              {{ element.name }}
            </div>

            <ul class="text-xs">
              <li class="cursor-pointer inline mr-4 text-sky-500 hover:text-blue-900" @click="renamePageModal(element)">Rename</li>
              <li class="cursor-pointer inline text-red-500 hover:text-red-900" @click="destroy(element)">Delete</li>
            </ul>
          </div>
        </div>

        <!-- modal to edit the page -->
        <form v-else class="bg-white border border-gray-200 hover:bg-slate-50 rounded-lg mb-2 item-list" @submit.prevent="update(element)">
          <div class="p-5 border-b border-gray-200">
            <errors :errors="form.errors" />

            <text-input :ref="'rename' + element.id"
                        v-model="form.name"
                        :label="'Name'" :type="'text'"
                        :autofocus="true"
                        :input-class="'block w-full'"
                        :required="true"
                        :autocomplete="false"
                        :maxlength="255"
                        @esc-key-pressed="renamePageModalShownId = 0"
            />
          </div>

          <div class="p-5 flex justify-between">
            <pretty-span :text="'Cancel'" :classes="'mr-3'" @click.prevent="renamePageModalShownId = 0" />
            <pretty-button :text="'Rename'" :state="loadingState" :icon="'check'" :classes="'save'" />
          </div>
        </form>
      </template>
    </draggable>

    <!-- modal to create a new page -->
    <form v-if="createPageModalShown" class="bg-white border border-gray-200 rounded-lg mb-6" @submit.prevent="submit()">
      <div class="p-5 border-b border-gray-200">
        <errors :errors="form.errors" />

        <text-input :ref="'newPage'"
                    v-model="form.name"
                    :label="'Name of the page'" :type="'text'"
                    :autofocus="true"
                    :input-class="'block w-full'"
                    :required="true"
                    :autocomplete="false"
                    :maxlength="255"
                    @esc-key-pressed="createPageModalShown = false"
        />
      </div>

      <div class="p-5 flex justify-between">
        <pretty-span :text="'Cancel'" :classes="'mr-3'" @click="createPageModalShown = false" />
        <pretty-button :text="'Add page'" :state="loadingState" :icon="'plus'" :classes="'save'" />
      </div>
    </form>

    <!-- blank state -->
    <div v-if="localPages.length == 0">
      <p class="p-5 text-center bg-white border border-gray-200 rounded-lg">Create at least one page to display contact's data.</p>
    </div>
  </div>
</template>

<script>
import PrettyButton from '@/Shared/Form/PrettyButton';
import PrettySpan from '@/Shared/Form/PrettySpan';
import TextInput from '@/Shared/Form/TextInput';
import Errors from '@/Shared/Form/Errors';
import draggable from 'vuedraggable';

export default {
  components: {
    PrettyButton,
    PrettySpan,
    TextInput,
    Errors,
    draggable,
  },

  props: {
    data: {
      type: Array,
      default: () => [],
    },
  },

  data() {
    return {
      loadingState: '',
      createPageModalShown: false,
      renamePageModalShownId: 0,
      localPages: [],
      isSelectedId: 0,
      drag: false,
      form: {
        name: '',
        position: '',
        errors: [],
      },
    };
  },

  mounted() {
    this.localPages = this.data.template_pages;
  },

  methods: {
    showPageModal() {
      this.form.errors = [];
      this.form.name = '';
      this.createPageModalShown = true;

      this.$nextTick(() => {
        this.$refs.newPage.focus();
      });
    },

    renamePageModal(page) {
      this.form.name = page.name;
      this.renamePageModalShownId = page.id;

      this.$nextTick(() => {
        this.$refs[`rename${page.id}`].focus();
      });
    },

    submit() {
      this.loadingState = 'loading';

      axios.post(this.data.url.template_page_store, this.form)
        .then(response => {
          this.flash('The page has been added', 'success');
          this.localPages.push(response.data.data);
          this.loadingState = null;
          this.createPageModalShown = false;
        })
        .catch(error => {
          this.loadingState = null;
          this.form.errors = error.response.data;
        });
    },

    update(page) {
      this.loadingState = 'loading';

      axios.put(page.url.update, this.form)
        .then(response => {
          this.flash('The page has been updated', 'success');
          this.localPages[this.localPages.findIndex(x => x.id === page.id)] = response.data.data;
          this.loadingState = null;
          this.renamePageModalShownId = 0;
        })
        .catch(error => {
          this.loadingState = null;
          this.form.errors = error.response.data;
        });
    },

    destroy(page) {
      if(confirm('Are you sure? This will remove the pages from all contacts, but won\'t delete the contacts themselves.')) {

        axios.delete(page.url.destroy)
          .then(response => {
            this.flash('The page has been deleted', 'success');
            var id = this.localPages.findIndex(x => x.id === page.id);
            this.localPages.splice(id, 1);
          })
          .catch(error => {
            this.loadingState = null;
            this.form.errors = error.response.data;
          });
      }
    },

    updatePosition(event) {
      // the event object comes from the draggable component
      this.form.position = event.moved.newIndex + 1;

      axios.post(event.moved.element.url.order, this.form)
        .then(response => {
          this.flash('The order has been saved', 'success');
        })
        .catch(error => {
          this.loadingState = null;
          this.errors = error.response.data;
        });
    },

    selectPage(page) {
      this.isSelectedId = page.id;
      this.$emit('pageSelected', page);
    }
  },
};
</script>