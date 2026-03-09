<template>
  <div class="card">
    <div class="card-header">
      <span>{{ card.title }}</span>
      <div class="card-actions" v-if="canEdit || canDelete">
        <button v-if="canEdit" @click="$emit('edit')" title="Редактировать">[Ред]</button>
        <button v-if="canDelete" @click="$emit('delete')" title="Удалить">[Уд]</button>
      </div>
    </div>
   
    <div class="card-description">{{ card.description }}</div>
   
    <div class="card-dates">
      <div class="date-item">
        <span>Создано:</span>
        <span>{{ card.createdAt }}</span>
      </div>
      <div v-if="card.editedAt" class="date-item">
        <span>Изменено:</span>
        <span>{{ card.editedAt }}</span>
      </div>
      <div class="date-item" style="margin-top:4px; font-weight:bold;">
        <span>Дедлайн:</span>
        <span :style="{ color: isOverdue ? 'red' : 'black' }">{{ formatDeadline(card.deadline) }}</span>
      </div>
    </div>
   
    <div v-if="card.status === 'done'" class="status-box" :class="{ overdue: card.isOverdue }">
      {{ card.isOverdue ? 'ПРОСРОЧЕНО' : 'ВЫПОЛНЕНО В СРОК' }}
    </div>
   
    <div v-if="card.returnReason" class="return-reason">
      <strong>Причина возврата:</strong> {{ card.returnReason }}
    </div>
   
    <div class="card-footer">
      <button
        v-if="columnId === 'planned'"
        @click="$emit('move-forward')"
        class="move-btn"
      >
        В работу ->
      </button>
     
      <button
        v-if="columnId === 'inProgress'"
        @click="$emit('move-forward')"
        class="move-btn"
      >
        На тестирование ->
      </button>
     
      <div v-if="columnId === 'testing'" style="display:flex; flex-direction:column; gap:5px;">
        <button @click="$emit('move-forward')" class="move-btn">
          Готово [ОК]
        </button>
        <button @click="$emit('return')" class="move-btn danger">
          <- Вернуть
        </button>
      </div>
    </div>
  </div>
</template>


<script>
import { computed } from 'vue';


export default {
  name: 'Card',
  props: {
    card: Object,
    columnId: String,
    canEdit: Boolean,
    canDelete: Boolean,
  },
  emits: ['edit', 'delete', 'move-forward', 'return'],
  setup(props) {
    const isOverdue = computed(() => {
      if (!props.card.deadline) return false;
      const deadline = parseDate(props.card.deadline);
      if (!deadline) return false;
      deadline.setHours(23, 59, 59, 999);
      return new Date() > deadline;
    });


    const parseDate = (dateString) => {
      if (!dateString) return null;
      try {
        if (dateString.includes('-') && dateString.length === 10) {
          const [year, month, day] = dateString.split('-');
          return new Date(year, month - 1, day);
        }
        if (dateString.includes('.')) {
          const parts = dateString.split(' ')[0].split('.');
          if (parts.length === 3) {
            return new Date(parts[2], parts[1] - 1, parts[0]);
          }
        }
        return null;
      } catch (e) {
        return null;
      }
    };


    const formatDeadline = (dateString) => {
      if (!dateString) return '-';
      const date = parseDate(dateString);
      if (!date || isNaN(date.getTime())) return 'Неверная дата';
      return date.toLocaleDateString('ru-RU');
    };


    return { isOverdue, formatDeadline };
  },
};
</script>