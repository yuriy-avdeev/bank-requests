<template>
  <ui-page-frame title="Заявка">
    <p><strong>Имя</strong>: {{ user.fio }}</p>
    <p><strong>Телефон</strong>: {{ user.phone }}</p>
    <p><strong>Сумма</strong>: {{ currency(user.amount) }}</p>
    <p>
      <strong>Статус</strong>:
      <UiRequestStatus :type="user.status ? user.status : status" />
    </p>

    <div class="col-5 mb-4 p-0">
      <label for="status"><strong>Выберите новый статус: </strong></label>
      <b-form-select size="sm" id="status" v-model="status" :options="options">
      </b-form-select>
    </div>

    <b-button class="mr-3" variant="outline-danger" @click="remove">
      Удалить
    </b-button>
    <b-button variant="outline-info" v-if="isChangesStatus" @click="update">
      Обновить
    </b-button>
  </ui-page-frame>
</template>

<script>
import { currency } from "../middleware/currency";

export default {
  data() {
    return {
      currency,
      user: {},
      status: "active",
      options: [
        { value: "active", text: "Активен" },
        { value: "cancelled", text: "Отменен" },
        { value: "done", text: "Завершен" },
        { value: "pending", text: "В работе" },
      ],
    };
  },

  mounted() {
    this.user = this.$store.getters.requests.find(
      (request) => request.id === this.$route.params.userId
    );
    this.status = this.user.status;
  },

  computed: {
    isChangesStatus() {
      return this.status !== this.user.status;
    },
  },

  methods: {
    async remove() {
      await this.$store.dispatch("remove", this.$route.params.userId);
      this.$router.push("/");
    },

    async update() {
      // отправить на сервер новый выбранный статус (связан через v-model)
      const { id, ...userData } = this.user; // - создал userData в формате для базы данных -> без id
      const data = { ...userData, status: this.status, id: id };
      await this.$store.dispatch("update", data);
      this.user.status = this.status; // если ответ - ок -> локально обновил тут
    },
  },
};
</script>
