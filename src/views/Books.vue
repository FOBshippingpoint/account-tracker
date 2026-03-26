<template>
  <div class="page-container">
    <BookList
      v-if="!selectedBookId"
      @select="openBook"
      @add="openNewBook"
    />
    <BookDetail
      v-else
      :bookId="selectedBookId"
      @back="selectedBookId = null"
      @edit="openEditBook"
      @settle="showSettlementSheet = true"
      @add-record="openNewRecord"
      @edit-record="openEditRecord"
    />

    <!-- ===== Modals & Sheets (Shared) ===== -->
    <CreateBookModal
      v-model="showCreateModal"
      :editBookId="editBookId"
      @created="(id) => (selectedBookId = id)"
    />
    <BookAddRecordSheet
      v-if="currentBook"
      v-model="showAddRecordSheet"
      :bookName="currentBook.name"
      :members="currentBook.members"
      :editRecordId="editRecordId"
    />
    <BookSettlementSheet
      v-model="showSettlementSheet"
      :bookName="currentBook?.name ?? ''"
      :memberStats="store.memberStats"
      :settlements="store.settlements"
    />
  </div>
</template>

<script setup lang="ts">
import { ref, computed } from "vue";
import { useTrackerStore } from "../stores/tracker";
import BookList from "../components/books/BookList.vue";
import BookDetail from "../components/books/BookDetail.vue";
import CreateBookModal from "../components/books/CreateBookModal.vue";
import BookAddRecordSheet from "../components/books/BookAddRecordSheet.vue";
import BookSettlementSheet from "../components/books/BookSettlementSheet.vue";

const store = useTrackerStore();

const selectedBookId = ref<string | null>(null);
const showCreateModal = ref(false);
const showAddRecordSheet = ref(false);
const showSettlementSheet = ref(false);
const editRecordId = ref<string | undefined>(undefined);
const editBookId = ref<string | undefined>(undefined);

const currentBook = computed(
  () => store.books.find((b) => b.id === selectedBookId.value) ?? null,
);

const openBook = (id: string) => {
  selectedBookId.value = id;
  store.selectBook(id);
};

const openNewBook = () => {
  editBookId.value = undefined;
  showCreateModal.value = true;
};

const openEditBook = () => {
  editBookId.value = selectedBookId.value ?? undefined;
  showCreateModal.value = true;
};

const openNewRecord = () => {
  editRecordId.value = undefined;
  showAddRecordSheet.value = true;
};

const openEditRecord = (id: string) => {
  editRecordId.value = id;
  showAddRecordSheet.value = true;
};
</script>
