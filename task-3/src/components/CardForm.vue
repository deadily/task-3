<template>
  <form @submit.prevent="submitForm" class="card-form">
    <div class="form-group">
      <label>Заголовок</label>
      <input v-model="form.title" required class="form-input" />
    </div>
    <div class="form-group">
      <label>Описание</label>
      <textarea v-model="form.description" required class="form-input" rows="3"></textarea>
    </div>
    <div class="form-group">
      <label>Дедлайн</label>
      <input
        v-model="form.deadline"
        type="date"
        required
        class="form-input"
      />
    </div>
    <div class="form-actions">
      <button type="button" @click="$emit('cancel')" class="btn-cancel">Отмена</button>
      <button type="submit" class="btn-submit">{{ isEdit ? 'Сохранить' : 'Создать' }}</button>
    </div>
  </form>
</template>


<script>
import { ref, computed, onMounted } from 'vue';


export default {
  props: { card: Object },
  emits: ['submit', 'cancel'],
  setup(props, { emit }) {
    const form = ref({ title: '', description: '', deadline: '' });
    const isEdit = computed(() => !!props.card);


    onMounted(() => {
      if (props.card && props.card.deadline) {
        if (props.card.deadline.includes('-') && props.card.deadline.length === 10) {
          form.value.deadline = props.card.deadline;
        } else {
          try {
            const parts = props.card.deadline.split(' ')[0].split('.');
            if (parts.length === 3) {
              form.value.deadline = `${parts[2]}-${parts[1]}-${parts[0]}`;
            }
          } catch (e) {
            form.value.deadline = '';
          }
        }
        form.value.title = props.card.title || '';
        form.value.description = props.card.description || '';
      }
    });


    const submitForm = () => {
      emit('submit', {
        ...form.value
      });
    };


    return { form, isEdit, submitForm };
  },
};
</script>
