<style scoped>
.fade-enter-active,
.fade-leave-active {
    transition: opacity .4s
}
</style>

<template>
  <div class="mw9 center">
    <!-- Left sidebar -->
    <div :class="[ dirltr ? 'fl' : 'fr' ]" class="w-70-ns w-100 pa2">
      <!-- How was your day -->
      <journal-rate-day @hasRated="hasRated" />

      <!-- Logs -->
      <div v-for="journalEntry in journalEntries.data" :key="journalEntry.id" class="cf" :cy-name="'entry-body-' + journalEntry.id">
        <journal-content-rate v-if="journalEntry.journalable_type == 'App\\Models\\Journal\\Day'" :journal-entry="journalEntry" @deleteJournalEntry="deleteJournalEntry" />

        <journal-content-activity v-else-if="journalEntry.journalable_type == 'App\\Models\\Contact\\Activity'" :journal-entry="journalEntry" />

        <journal-content-entry v-else-if="journalEntry.journalable_type == 'App\\Models\\Journal\\Entry'" :journal-entry="journalEntry" @deleteJournalEntry="deleteJournalEntry" />
      </div>

      <div v-if="(journalEntries.per_page * journalEntries.current_page) <= journalEntries.total" class="br3 ba b--gray-monica bg-white pr3 pb3 pt3 mb3 tc">
        <p class="mb0 pointer" @click="loadMore()">
          <span v-if="!loadingMore">
            {{ $t('app.load_more') }}
          </span>
          <span v-else class="black-50">
            {{ $t('app.loading') }}
          </span>
        </p>
      </div>

      <div v-if="journalEntries.total === 0" class="br3 ba b--gray-monica bg-white pr3 pb3 pt3 mb3 tc" cy-name="journal-blank-state">
        <div class="tc mb4">
          <img src="/img/journal/blank.svg" />
        </div>
        <h3>
          {{ $t('journal.journal_blank_cta') }}
        </h3>
        <p>{{ $t('journal.journal_blank_description') }}</p>
      </div>
    </div>

    <!-- Right sidebar -->
    <div :class="[ dirltr ? 'fl' : 'fr' ]" class="w-30 pa2">
      <a href="/journal/add" class="btn btn-primary w-100 mb4" cy-name="add-entry-button">
        {{ $t('journal.journal_add') }}
      </a>
      <p>{{ $t('journal.journal_description') }}</p>
    </div>
  </div>
</template>

<script>
export default {
  data() {
    return {
      journalEntries: [],

      day: {
        rate: ''
      },

      showSadSmileyColor: false,
      showHappySmileyColor: false,
      loadingMore: false,

      dirltr: true,
    };
  },

  computed: {
    hasMorePage: function() {
      var total = this.journalEntries.per_page * this.journalEntries.current_page;

      if (total >= this.journalEntries.total) {
        return true;
      }

      return false;
    }
  },

  mounted() {
    this.prepareComponent();
  },

  methods: {
    prepareComponent() {
      this.dirltr = this.$root.htmldir == 'ltr';
      this.getEntries();
    },

    getEntries() {
      axios.get('/journal/entries')
        .then(response => {
          this.journalEntries = response.data;
          this.journalEntries.current_page = response.data.current_page;
          this.journalEntries.next_page_url = response.data.next_page_url;
          this.journalEntries.per_page = response.data.per_page;
          this.journalEntries.prev_page_url = response.data.prev_page_url;
          this.journalEntries.total = response.data.total;
        });
    },

    // This event is omited from the child component
    deleteJournalEntry: function($journalEntryId) {
      // check if the deleted entry date is today. If that's the case
      // we need to put back the Rate box. This is only necessary if
      // the user does all his actions on the same page without ever
      // reloading the page.
      this.journalEntries.data.filter(function(obj) {
        return obj.id == $journalEntryId;
      });

      // Filter out the array without the deleted Journal Entry
      this.journalEntries.data = this.journalEntries.data.filter(function(element) {
        return element.id !== $journalEntryId;
      });
    },

    hasRated: function(journalObject) {
      this.journalEntries.data.unshift(journalObject);
    },

    loadMore() {
      this.loadingMore = true;
      axios.get('/journal/entries?page=' + (this.journalEntries.current_page + 1))
        .then(response => {
          this.journalEntries.current_page = response.data.current_page;
          this.journalEntries.next_page_url = response.data.next_page_url;
          this.journalEntries.per_page = response.data.per_page;
          this.journalEntries.prev_page_url = response.data.prev_page_url;
          this.journalEntries.total = response.data.total;

          for (var j of response.data.data) {
            this.journalEntries.data.push(j);
          }

          this.loadingMore = false;
        });
    },
  }
};
</script>
