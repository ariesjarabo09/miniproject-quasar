<template>
  <q-page class="flex flex-center">
    <q-card class="my-card" style="width: 600px">
      <q-card-section>
        <div class="text-h4">To-Do List</div>
      </q-card-section>

      <q-card-section>
        <!-- Input for adding a new task -->
        <div class="row justify-center">
          <div class="col-12">
            <q-input v-model="todoTask" label="Input Task" />
            <q-input
              v-model="personel"
              label="Assigned Personnel To The Task"
            />
            <q-btn
              round
              dense
              flat
              label="Submit"
              color="primary"
              :disable="!todoTask"
              @click="addItem"
            />
          </div>
        </div>
      </q-card-section>

      <q-separator />

      <q-card-section v-if="todoList.length > 0">
        <!-- Displaying the list of tasks -->
        <q-table
          :rows="todoList"
          :columns="columns"
          row-key="id"
          flat
          wrap-cells
          class="my-table"
        >
          <template v-slot:body-cell-todoList="props">
            <q-td v-if="!props.row.editing" class="text-left">{{
              props.row.todoList
            }}</q-td>
            <q-td v-else class="text-left">
              <q-input
                v-model="props.row.todoList"
                dense
                @blur="saveItem(props.row)"
                @keydown.enter="saveItem(props.row)"
              />
            </q-td>
          </template>

          <template v-slot:body-cell-name="props">
            <q-td v-if="!props.row.editing" class="text-left">{{
              props.row.name
            }}</q-td>
            <q-td v-else class="text-left">
              <q-input
                v-model="props.row.name"
                dense
                @blur="saveItem(props.row)"
                @keydown.enter="saveItem(props.row)"
              />
            </q-td>
          </template>

          <template v-slot:body-cell-actions="props">
            <q-td key="actions" class="text-center">
              <q-btn
                round
                dense
                flat
                icon="edit"
                class="text-light-blue-6"
                @click="toggleEditing(props.row)"
                v-if="!props.row.editing"
              />
              <q-btn
                round
                dense
                flat
                icon="done"
                class="text-light-blue-6"
                @click="toggleEditing(props.row)"
                v-else
              />
              <q-btn
                round
                dense
                flat
                icon="clear"
                class="text-red-6"
                @click="confirmDeleteItem(props.row)"
              />
            </q-td>
          </template>
        </q-table>
      </q-card-section>

      <q-card-section v-else>
        <!-- Message displayed when no items in the list -->
        <div class="row justify-center">
          <div class="col-12">
            <div class="text-h6">No Items</div>
          </div>
        </div>
      </q-card-section>

      <!-- Confirmation dialog -->
      <q-dialog v-model="showConfirmation" persistent>
        <q-card>
          <q-card-section>
            <div class="text-h6">
              Are you sure you want to delete this task?
            </div>
          </q-card-section>
          <q-card-actions align="right">
            <q-btn label="Cancel" color="primary" @click="cancelDeleteItem" />
            <q-btn
              label="Delete"
              color="negative"
              @click="deleteItemConfirmed"
            />
          </q-card-actions>
        </q-card>
      </q-dialog>
    </q-card>
  </q-page>
</template>

<script>
import { defineComponent, ref, onMounted } from "vue";

export default defineComponent({
  name: "IndexPage",

  setup() {
    // Task list and task input variables
    const todoTask = ref(""); // Define todoTask variable
    const todoList = ref([]); // Initialize with an empty array
    const showConfirmation = ref(false); // Controls the display of the confirmation dialog
    const itemToDelete = ref(null); // Holds the reference to the item being deleted
    const personel = ref("");
    const personelList = ref([]);

    // Load tasks from local storage on component mount
    onMounted(() => {
      fetchTasksFromLocalStorage();
    });

    function fetchTasksFromLocalStorage() {
      const storedTasks = localStorage.getItem("todoList");
      if (storedTasks) {
        todoList.value = JSON.parse(storedTasks);
        console.log("Fetched tasks:", todoList.value);
      }
    }

    // Add a new task to the list
    function addItem() {
      if (!todoTask.value) {
        return; // Don't add an empty task
      }

      const newTask = {
        id: Date.now(),
        todoList: todoTask.value,
        name: personel.value,
        editing: false,
      };

      todoList.value.push(newTask);

      // Clear input fields
      todoTask.value = "";
      personel.value = "";

      saveTasksToLocalStorage(); // Save tasks to local storage
    }

    // Remove a task from the list
    function deleteItem(item) {
      const index = todoList.value.findIndex((task) => task.id === item.id);
      if (index !== -1) {
        todoList.value.splice(index, 1);
        saveTasksToLocalStorage(); // Save tasks to local storage
      }
    }

    // Confirm deletion of a task
    function confirmDeleteItem(item) {
      itemToDelete.value = item;
      showConfirmation.value = true;
    }

    // Cancel deletion of a task
    function cancelDeleteItem() {
      showConfirmation.value = false;
      itemToDelete.value = null;
    }

    // Delete the task after confirmation
    function deleteItemConfirmed() {
      deleteItem(itemToDelete.value);
      showConfirmation.value = false;
      itemToDelete.value = null;
    }

    // Toggle task editing mode
    function toggleEditing(item) {
      item.editing = !item.editing;
      console.log(item.editing);
    }

    // Save the tasks to local storage
    function saveTasksToLocalStorage() {
      localStorage.setItem("todoList", JSON.stringify(todoList.value));
    }

    // Save the edited task
    function saveItem(item) {
      item.editing = false;
      saveTasksToLocalStorage(); // Save tasks to local storage
    }

    // Define the table columns
    const columns = [
      {
        name: "todoList",
        required: true,
        label: "Task",
        align: "left",
        field: "todoList",
        sortable: true,
      },
      {
        name: "name",
        required: true,
        label: "Assigned Personnel",
        align: "left",
        field: "name",
        sortable: true,
      },
      { name: "actions", label: "Actions", field: "actions", align: "center" },
    ];

    return {
      todoTask,
      todoList,
      showConfirmation,
      itemToDelete,
      personel,
      personelList,
      columns,
      addItem,
      deleteItem,
      confirmDeleteItem,
      cancelDeleteItem,
      deleteItemConfirmed,
      toggleEditing,
      saveItem,
    };
  },
});
</script>

<style>
.my-card {
  max-width: 600px;
  margin: auto;
}

.my-table {
  width: 100%;
}

.text-h4 {
  font-size: 24px;
}

.text-h6 {
  font-size: 18px;
}

.row {
  margin-bottom: 15px;
}

.row:last-child {
  margin-bottom: 0;
}
</style>
