<template>
  <div class="column">
    <div class="column-header">
      <div class="column-title">
        <span>{{ title }}</span>
      </div>
      <div class="column-counts">
        <span class="total-count">{{ cards.length }}</span>
        <span v-if="overdueCount > 0" class="overdue-count">{{ overdueCount }} просроч.</span>
      </div>
    </div>
    <div class="card-list">
      <Card
        v-for="card in cards" :key="card.id"
        :card="card" :column-id="columnId"
        @edit="$emit('edit-card', card)"
        @delete="$emit('delete-card', card.id)"
        @move-forward="$emit('move-card', card, nextColumn)"
        @return="$emit('return-card', card)"
        :can-edit="canEdit" :can-delete="canDelete"
      />
    </div>
    <button v-if="canAdd" @click="$emit('add-card', null)" class="add-card-btn">
      + Добавить
    </button>
  </div>
</template>



<script>
import Card from './Card.vue';
export default {
  components: { Card },
  props: {
    title: String, 
    cards: Array, 
    columnId: String,
    overdueCount: Number,
    canAdd: Boolean, 
    canEdit: Boolean, 
    canDelete: Boolean,
    nextColumn: String, 
    prevColumn: String,
  },
  emits: ['add-card', 'move-card', 'edit-card', 'delete-card', 'return-card'],
};
</script>
