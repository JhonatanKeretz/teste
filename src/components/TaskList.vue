<template>
  <v-simple-table>
    <template v-slot:default>
      <v-data-table
        :headers="headers"
        :items="desserts"
        sort-by="calories"
        class="elevation-1"
      >
        <template v-slot:top>
          <v-toolbar flat>
            <v-toolbar-title>Plano de Ação</v-toolbar-title>
            <v-divider class="mx-4" inset vertical></v-divider>
            <v-spacer></v-spacer>
            <v-dialog v-model="dialog" max-width="1500px">
              <template v-slot:activator="{ on, attrs }">
                <v-btn
                  color="primary"
                  dark
                  class="mb-2"
                  v-bind="attrs"
                  v-on="on"
                >
                  New Item
                </v-btn>
              </template>
              <v-card>
                <v-card-title>
                  <span class="text-h5">{{ formTitle }}</span>
                </v-card-title>

                <v-card-text>
                  <v-container>
                    <v-row>
                      <v-col cols="8" sm="8">
                        <v-text-field
                          v-model="title"
                          label="O que será feito?"
                        ></v-text-field>
                      </v-col>
                      <v-col cols="8" sm="12">
                        <v-textarea
                          v-model="why"
                          label="Por que será feito? Quais Resultados esperados?"
                        ></v-textarea>
                      </v-col>
                    </v-row>
                    <v-row>
                      <v-col cols="12" sm="6">
                        <v-text-field
                          v-model="when"
                          label="Quando será feito?"
                        ></v-text-field>
                      </v-col>
                      <v-col cols="12" sm="6">
                        <v-select
                          v-model="where"
                          label="Onde será feito?"
                          :items="area"
                        ></v-select>
                      </v-col>

                      <v-col cols="12" sm="6">
                        <v-select
                          v-model="who"
                          label="Por quem será feito?"
                          :items="items"
                        ></v-select>
                      </v-col>
                    </v-row>
                    <v-row>
                      <v-col cols="12" sm="6">
                        <v-textarea
                          v-model="who"
                          label="Como será feito?"
                        ></v-textarea>
                      </v-col>
                    </v-row>
                    <v-row>
                      <v-col cols="12" sm="6">
                        <v-select
                          v-model="howmuch"
                          label="Quando vai custar? Qual será o esforço?"
                          :items="valor"
                        ></v-select>
                      </v-col>
                      <v-col cols="12" sm="6">
                        <v-text-field
                          v-model="howmuchValor"
                          label="*aqui será o valor add exemplo"
                        ></v-text-field>
                      </v-col>
                    </v-row>
                    <v-row>
                      <v-col cols="12" sm="6">
                        <v-select
                          v-model="priority"
                          label="Prioridade"
                          :items="prioridade"
                        ></v-select>
                      </v-col>
                    </v-row>
                  </v-container>
                </v-card-text>

                <v-card-actions>
                  <v-spacer></v-spacer>
                  <v-btn color="blue darken-1" text @click="close">
                    Cancel
                  </v-btn>
                  <v-btn color="blue darken-1" text @click="saveTask()">
                    Save
                  </v-btn>
                </v-card-actions>
              </v-card>
            </v-dialog>
            <v-dialog v-model="dialogDelete" max-width="500px">
              <v-card>
                <v-card-title class="text-h5"
                  >Você quer deletar esta Ação?</v-card-title
                >
                <v-card-actions>
                  <v-spacer></v-spacer>
                  <v-btn color="blue darken-1" text @click="closeDelete"
                    >Cancel</v-btn
                  >
                  <v-btn color="blue darken-1" text @click="deleteItemConfirm"
                    >OK</v-btn
                  >
                  <v-spacer></v-spacer>
                </v-card-actions>
              </v-card>
            </v-dialog>
          </v-toolbar>
        </template>
        <template v-slot:item.actions="{ item }">
          <v-icon small class="mr-2" @click="editItem(item)">
            mdi-pencil
          </v-icon>
          <v-icon small @click="deleteItem(item)"> mdi-delete </v-icon>
        </template>
        <template v-slot:no-data>
          <v-btn color="primary" @click="initialize"> Reset </v-btn>
        </template>
      </v-data-table>

      
      <tbody>
        <tr
          class="text-left"
          style="cursor: pointer"
          v-for="(task, index) in tasks"
          :key="index"
          @click="goTo(`/tasks/${task._id}`)"
        >
          <td>
            {{ task.title }}
          </td>

          <td>
            {{ task.priority }}
          </td>
          <v-divider></v-divider>
        </tr>
      </tbody>
    </template>
  </v-simple-table>
</template>

<script lang="ts">
import { Task } from "@/interfaces/Task";
import { createTask, getTasks } from "@/services/TaskService";
import { defineComponent } from "vue";
import { RawLocation } from "vue-router";

export default defineComponent({
  data() {
    return {
      formTitle: "Criar Plano de Ação",
      dialog: false,
      dialogDelete: false,
      headers: [
        {
          text: "Titulo da Ação",
          align: "start",
          sortable: false,
          value: "name",
        },
        { text: "Descrição", value: "description" },

        { text: "Ação(Prioridade)", value: "actions", sortable: false },
      ],
      // ----------
      desserts: [],
      editedIndex: -1,
      editedItem: {
        title: "",
        why: "",
        when: "",
        where: "",
        who: "",
        how: "",
        howmuch: "",
        howmuchValor: "",
        priority: "",
        description: "",
      },
      defaultItem: {
        title: "",
        why: "",
        when: "",
        where: "",
        who: "",
        how: "",
        howmuch: "",
        howmuchValor: "",
        priority: "",
        description: "",
      },
      computed: {
        formTitle() {
          return this.editedIndex === -1 ? "New Item" : "Edit Item";
        },
      },

      watch: {
        dialog(val) {
          val || this.close();
        },
        dialogDelete(val) {
          val || this.closeDelete();
        },
      },
      created() {
        this.initialize();
      },

      // ---------------------
      items: ["Rafael", "Jhonatan", "Jose", "Carlos"],
      checkbox: false,
      area: [
        "Administrativo",
        "Comercial",
        "Engenharia",
        "Financeiro",
        "Logística",
        "Marketing",
        "Produção",
        "Produto",
        "RH",
      ],
      valor: ["Horas", "Valor"],
      prioridade: ["Alta", "Média", "Baixa"],

      tasks: {} as Task[],
    };
  },
  methods: {
    async loadTasks() {
      const res = await getTasks();
      this.tasks = res.data;
    },
    async saveTask() {
      const res = await createTask(this.task);
      console.log(res);

      this.$router.push({ name: "task" });
    },
    goTo(route: RawLocation) {
      this.$router.push(route);
    },
  },
  mounted() {
    this.loadTasks();
  },
});
</script>

<style></style>
