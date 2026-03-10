<template>
  <div class="board">
    <Column
      title="Запланированные"
      :cards="columns.planned"
      column-id="planned"
      @add-card="openEditModal"
      @move-card="handleMoveCard"
      @edit-card="openEditModal"
      @delete-card="handleDeleteCard"
      :can-add="true" :can-edit="true" :can-delete="true"
      next-column="inProgress"
    />
    <Column
      title="В работе"
      :cards="columns.inProgress"
      column-id="inProgress"
      @move-card="handleMoveCard"
      @edit-card="openEditModal"
      @delete-card="handleDeleteCard"
      :can-add="false" :can-edit="true" :can-delete="true"
      next-column="testing"
    />
    <Column
      title="Тестирование"
      :cards="columns.testing"
      column-id="testing"
      @move-card="handleMoveCard"
      @edit-card="openEditModal"
      @return-card="openReturnModal"
      :can-add="false" :can-edit="true" :can-delete="false"
      next-column="done"
    />
    <Column
      title="Готовые"
      :cards="columns.done"
      :overdue-count="overdueDoneCount"
      column-id="done"
      :can-add="false" :can-edit="false" :can-delete="false"
    />
   
    <div v-if="showEditModal" class="modal-overlay" @click="closeModals">
      <div class="modal" @click.stop>
        <h3>{{ isEditing ? 'Редактировать задачу' : 'Новая задача' }}</h3>
        <CardForm
          :card="editingCard"
          @submit="saveCard"
          @cancel="closeModals"
        />
      </div>
    </div>
   
    <div v-if="showReturnModal" class="modal-overlay" @click="closeModals">
      <div class="modal" @click.stop>
        <h3>Возврат в работу</h3>
        <div class="form-group">
          <label>Причина</label>
          <textarea
            v-model="returnReason"
            class="form-input"
            rows="3"
          ></textarea>
        </div>
        <div class="modal-actions">
          <button @click="closeModals" class="btn-cancel">Отмена</button>
          <button @click="confirmReturn" class="btn-confirm">Вернуть</button>
        </div>
      </div>
    </div>
  </div>
</template>

<script>
import Column from './Column.vue'
import CardForm from './CardForm.vue'
import { reactive, onMounted, watch, ref, computed } from 'vue'

export default {
  name: 'Board',
  components: { Column, CardForm },
  setup() {
    const columns = reactive({ planned: [], inProgress: [], testing: [], done: [] })
    const showEditModal = ref(false)
    const showReturnModal = ref(false)
    const editingCard = ref(null)
    const returningCard = ref(null)
    const returnReason = ref('')
    const isEditing = ref(false)


    const isCardOverdue = (card) => {
      if (!card.deadline) return false
      if (card.status === 'done') return card.isOverdue
      
      let deadline
      if (card.deadline.includes('-')) {
        const [year, month, day] = card.deadline.split('-')
        deadline = new Date(year, month - 1, day)
      } else {
        deadline = new Date(card.deadline)
      }
      deadline.setHours(23, 59, 59, 999)
      return new Date() > deadline
    }

    const overduePlannedCount = computed(() => 
      columns.planned.filter(card => isCardOverdue(card)).length
    )
    
    const overdueInProgressCount = computed(() => 
      columns.inProgress.filter(card => isCardOverdue(card)).length
    )
    
    const overdueTestingCount = computed(() => 
      columns.testing.filter(card => isCardOverdue(card)).length
    )
    
    const overdueDoneCount = computed(() => 
      columns.done.filter(card => card.isOverdue).length
    )

    const loadState = () => {
      const saved = localStorage.getItem('kanbanState')
      if (saved) {
        try {
          const state = JSON.parse(saved)
          Object.keys(columns).forEach(key => columns[key] = state[key] || [])
        } catch (e) { console.error(e) }
      }
    }

    const saveState = () => localStorage.setItem('kanbanState', JSON.stringify(columns))

    const handleAddCard = (cardData) => {
      const newCard = {
        id: Date.now(),
        title: cardData.title,
        description: cardData.description,
        deadline: cardData.deadline,
        createdAt: new Date().toLocaleString('ru-RU'),
        editedAt: null,
        status: 'planned',
        returnReason: null,
        isOverdue: false
      }
      columns.planned.push(newCard)
      saveState()
    }

    const openEditModal = (card) => {
      editingCard.value = card ? { ...card } : null
      isEditing.value = !!card
      showEditModal.value = true
    }

    const openReturnModal = (card) => {
      returningCard.value = card
      returnReason.value = ''
      showReturnModal.value = true
    }

    const closeModals = () => {
      showEditModal.value = false
      showReturnModal.value = false
      editingCard.value = null
      returningCard.value = null
      returnReason.value = ''
    }

    const saveCard = (cardData) => {
      if (isEditing.value && editingCard.value) {
        const allColumns = Object.values(columns)
        for (const col of allColumns) {
          const index = col.findIndex(c => c.id === editingCard.value.id)
          if (index !== -1) {
            const target = col[index]
            target.title = cardData.title
            target.description = cardData.description
            target.deadline = cardData.deadline // Сохраняем как YYYY-MM-DD
            target.editedAt = new Date().toLocaleString('ru-RU')
            break
          }
        }
      } else {
        handleAddCard(cardData)
      }
      closeModals()
      saveState()
    }

    const handleDeleteCard = (cardId) => {
      if (!confirm('Удалить задачу?')) return
      Object.values(columns).forEach(col => {
        const index = col.findIndex(c => c.id === cardId)
        if (index !== -1) col.splice(index, 1)
      })
      saveState()
    }

    const handleMoveCard = (card, newColumnId) => {
      let oldColKey = null
      for (const key in columns) {
        if (columns[key].some(c => c.id === card.id)) {
          oldColKey = key
          break
        }
      }
      if (oldColKey) columns[oldColKey] = columns[oldColKey].filter(c => c.id !== card.id)

      if (newColumnId === 'done') {
        let deadline;
        if (card.deadline.includes('-')) {
          const [year, month, day] = card.deadline.split('-');
          deadline = new Date(year, month - 1, day);
        } else {
          deadline = new Date(card.deadline);
        }
        deadline.setHours(23, 59, 59, 999);
        card.isOverdue = new Date() > deadline;
        card.completedAt = new Date().toLocaleString('ru-RU')
      }


      card.status = newColumnId
      columns[newColumnId].push(card)
      saveState()
    }

    const confirmReturn = () => {
      if (!returnReason.value.trim()) { alert('Укажите причину'); return }
      if (returningCard.value) {
        returningCard.value.returnReason = returnReason.value
        returningCard.value.returnedAt = new Date().toLocaleString('ru-RU')
        returningCard.value.isOverdue = false
        handleMoveCard(returningCard.value, 'inProgress')
      }
      closeModals()
    }

    onMounted(() => {
      loadState()
    })
   
    watch(columns, saveState, { deep: true })

    return {
      columns, 
      showEditModal, 
      showReturnModal, 
      editingCard, 
      returnReason, 
      isEditing,
      overduePlannedCount,
      overdueInProgressCount,
      overdueTestingCount,
      overdueDoneCount,
      openEditModal, 
      openReturnModal, 
      closeModals, 
      saveCard, 
      handleDeleteCard,
      handleMoveCard, 
      handleAddCard, 
      confirmReturn
    }
  }
}
</script>
