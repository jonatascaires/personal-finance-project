<template>
  <div
    class="
      my-4
      space-y-4
      border-2 border-indigo-200 border-dashed
      bg-indigo-50
      p-5
      rounded-xl
    "
  >
    <div class="grid sm:grid-cols-2 md:grid-cols-4 gap-5">
      <div>
        <AppFormLabel>Data da transação</AppFormLabel>
        <AppFormInput type="date" v-model="form.date" />
      </div>

      <div>
        <AppFormLabel>Valor</AppFormLabel>
        <AppFormInput v-model="form.amount" type="number" />
      </div>

      <div>
        <AppFormLabel>Descrição</AppFormLabel>
        <AppFormInput v-model="form.description" />
      </div>

      <div>
        <AppFormLabel>Categoria</AppFormLabel>
        <AppFormSelect v-model="form.categoryId" :options="categories" />
      </div>
    </div>

    <div class="space-x-4 flex items-center justify-end">
      <a href="" class="inline-flex text-gray-700 text-sm" @click.stop.prevent="onCancel">
        Cancelar
      </a>

      <AppButton @click="addTransacions"> Adicionar </AppButton>
    </div>
  </div>
</template>

<script>
import AppButton from "~/components/Ui/AppButton";
import AppFormInput from "~/components/Ui/AppFormInput";
import AppFormLabel from "~/components/Ui/AppFormLabel";
import AppFormSelect from "~/components/Ui/AppFormSelect";

export default {
  name: "TransactionAdd",

  components: {
    AppButton,
    AppFormInput,
    AppFormLabel,
    AppFormSelect,
  },

  data() {
    return {
      form: {
        date: "",
        amount: 0,
        description: "",
        categoryId: 1,
      },
      categories: [],
    };
  },
  async fetch() {
    this.categories = await this.$store.dispatch("categories/getCategories");
  },
  methods: {
    addTransacions() {
      if (!this.form.date || !this.form.description || this.form.amount == 0) {
        return;
      }
      this.$store
        .dispatch("transactions/addTransactions", this.form)
        .then((response) => {
          this.$emit("after-add", {
            ...response,
            category: this.categories.find((o) => o.id == this.form.categoryId),
          });
        });
    },
    onCancel() {
      this.$emit("cancel");
    },
  },
};
</script>