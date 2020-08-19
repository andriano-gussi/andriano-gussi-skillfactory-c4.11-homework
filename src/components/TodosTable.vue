<template>
  <div class="container">
    <div class="col-sm-10">
      <h1>Задачи</h1>
      <!--Кмопонент для отображения алертов-->
      <Confirmation
        :message="confirmationMessage"
        :showDismissibleAlert="showConfirmation"
        @showAlert="onShowAlert"/>
      <!--Кнопка добавления новой задачи-->
      <button type="button"
          id="task-add"
          class="btn btn-success btn-sm align-left d-block"
          v-b-modal.todo-modal>
        Добавить задачу
      </button>
      <!--Информация о количестчве выполненных и невыполненных задач-->
      <div class="text-left">
        <p class="mb-0">Выполненных задач: {{ complited }}</p>
        <p>Осталось выполнить: {{ notcomplited }}</p>
      </div>
      <!--Таблица задач-->
      <table class="table table-dark table-stripped table-hover">
        <thead class="thead-light">
          <tr>
            <th>Uid</th>
            <th>Описание</th>
            <th>Выполнена?</th>
            <th></th>
          </tr>
        </thead>

        <tbody>
          <tr v-for="(todo, index) in todos" :key="index">
            <td class="todo-uid">{{ todo.uid }}</td>
            <td>{{ todo.description }}</td>
            <td>
              <span v-if="todo.is_completed">Выполнено</span>
              <span v-else>Не выполнено</span>
            </td>
            <td>
              <div class="btn-group" role="group">
                <button type="button"
                  class="btn btn-secondary btn-sm"
                  v-b-modal.todo-update-modal
                  @click="updateTodo(todo)"
                  >
                  Обновить
                </button>
                &nbsp;
                <button type="button"
                  class="btn btn-danger btn-sm"
                  v-b-modal.delete-alarm-modal
                  @click="copyTodo=todo"
                  >
                  X
                </button>
              </div>
            </td>
          </tr>
        </tbody>

      </table>
      <!--Модальное окно, возникающее, когда добавляется новая задача-->
      <b-modal ref="addTodoModal"
        id="todo-modal"
        title="Добавить задачу"
        hide-footer>
        <b-form @submit="onSubmit" @reset="onReset" class="w-100">
          <b-form-group id="form-description-group"
            label="Описание:"
            label-for="form-description-input">
            <b-form-input id="form-description-input"
              type="text"
              v-model="addTodoForm.description"
              required
              placeholder="Завести задачу">
            </b-form-input>
          </b-form-group>
          <b-form-group id="form-complete-group">
            <b-form-checkbox-group v-model="addTodoForm.is_completed" id="form-checks">
              <b-form-checkbox value="true">Задача выполнена?</b-form-checkbox>
            </b-form-checkbox-group>
          </b-form-group>
          <b-button type="submit" variant="primary">Добавить</b-button>
          <b-button type="reset" variant="danger">Сброс</b-button>
        </b-form>
      </b-modal>
      <!--Модальное окно при обновлении задачи-->
      <b-modal ref="updateTodoModal"
        id="todo-update-modal"
        title="Update"
        hide-footer>
        <b-form @submit="onUpdateSubmit" @reset="onUpdateReset" class="w-100">
          <b-form-group id="form-update-description-group"
            label="Описание:"
            label-for="form-update-description-input">
            <b-form-input id="form-update-description-input"
              type="text"
              v-model="updateTodoForm.description"
              required>
            </b-form-input>
          </b-form-group>
          <b-form-group id="form-update-complete-group">
            <b-form-checkbox-group v-model="updateTodoForm.is_completed" id="form-update-checks">
              <b-form-checkbox value="true">Задача выполнена?</b-form-checkbox>
            </b-form-checkbox-group>
          </b-form-group>
          <b-button-group>
            <b-button type="submit" variant="primary">Обновить</b-button>
            <b-button type="reset" variant="danger">Сброс</b-button>
          </b-button-group>
        </b-form>
        </b-modal>
        <!--Модальное окно для подтверждения удаления задачи-->
        <b-modal  @ok="deleteTodo(copyTodo)"
          @reset="onUpdateReset"
          id = "delete-alarm-modal"
          ok-variant="danger"
          cancel-variant="primary"
          header-bg-variant="danger"
          centered
          title = "Внимание!"> 
          <p  class = "my-4 ">Вы собираетесь удалить задачу! Подтвердите свои действия</p> 
        </b-modal>

    </div>
  </div>
</template>

<script>
import Confirmation from './Confirmation.vue';

export default {
  name: 'TodosTable',
  data() {
    return {
      todos: [], //рабочий массив для всех задач
      copyTodo: {}, //копия для одной задачи
      lengthTodos: 0, //контроль общего количества задач
      complited: 0, //количество выполненных задач
      notcomplited: 0, //количество невыполненных задач
      //первоначальное состояние формы для новой задачи
      addTodoForm: {
        description: '',
        is_completed: [],
      },
      //для очитски формы
      updateTodoForm: {
        uid: 0,
        description: '',
        is_completed: [],
      },
      confirmationMessage: '', //сообщение для алерта
      showConfirmation: false, //отображать алерт, если true
    }
  },
  methods: {
    // формирует массив (todos) задач, хранящихся в localStorage,
    // подсчитывает количество выполненных и невыполненных задач 
    getTodos() {
      let keys = Object.keys(localStorage);
      keys.sort();
      this.lengthTodos = Number(localStorage.getItem('lengthTodos'));
      for (let i=0; i < this.lengthTodos; i +=1) {
        let receivedTask = JSON.parse(localStorage.getItem(keys[i]));
        receivedTask.uid = i + 1;
        if (receivedTask.is_completed) {
          this.complited +=1;
        }
        this.todos.push(receivedTask);
      }
      this.notcomplited = this.lengthTodos - this.complited;
    },
    // обнуляет необходимые параметры
    resetForm() {
      this.addTodoForm.description = '';
      this.addTodoForm.is_completed = [];
      this.updateTodoForm.description = '';
      this.updateTodoForm.is_completed = [];
      this.todos = [];
      this.complited = 0;
      this.notcomplited = 0;
    },
    // для отображения алерта
    onShowAlert() {
      this.showConfirmation = !this.showConfirmation
    },
    // проверяет совпадает ли описание новой задачи с уже имеющимися
    checkNewTask(taskDescription) {
      let result = false;
      this.todos.forEach(item => {
        if (item.description === taskDescription) {
          result = true;
        }
      });
      return result
    },
    // добавляет новую задачу, если такой еще не было
    onSubmit(event) {
      event.preventDefault();
      this.$refs.addTodoModal.hide();
      if (this.checkNewTask(this.addTodoForm.description)) {
        this.confirmationMessage = "Такакя задача уже есть! Введите другую.";
      } else {
        this.lengthTodos = Number(localStorage.getItem('lengthTodos')) + 1;
        localStorage.setItem('lengthTodos', this.lengthTodos);
        let newTask = {
          uid: this.lengthTodos,
          description: this.addTodoForm.description,
          is_completed: this.addTodoForm.is_completed.length > 0,
        };
        newTask = JSON.stringify(newTask);
        localStorage.setItem(this.lengthTodos, newTask);
        this.confirmationMessage = `Задача "${this.addTodoForm.description}" добавлена`;
      }
      this.showConfirmation = true;
      this.resetForm();
      this.getTodos();
    },
    // сброс окна добавления задачи
    onReset(event) {
      event.preventDefault();
      this.$refs.addTodoModal.hide();
      this.resetForm();
      this.getTodos();
    },
    // обновляет описание и состояние задачи
    updateTodo(todo) {
      this.updateTodoForm.uid = todo.uid;
      this.updateTodoForm.description = todo.description;
      if (todo.is_completed) {
        this.updateTodoForm.is_completed = [true];
      }
    },
    // срабатывает при нажатии на кнопку "Обновить" в окне обновления задачи
    onUpdateSubmit(event) {
      event.preventDefault();
      this.$refs.updateTodoModal.hide();
      let updatedTask = {
        uid: this.updateTodoForm.uid,
        description: this.updateTodoForm.description,
        is_completed: this.updateTodoForm.is_completed.length > 0,
      };
      updatedTask = JSON.stringify(updatedTask);
      localStorage.setItem(this.updateTodoForm.uid, updatedTask);
      this.confirmationMessage = `Задача "${this.updateTodoForm.description}" обновлена`;
      this.showConfirmation = true;
      this.resetForm();
      this.getTodos();
    },
    // срабатывает при нажатии на кнопку "Сбросить" в окне обновления задачи
    onUpdateReset(event) {
      event.preventDefault();
      this.$refs.updateTodoModal.hide();
    },
    // удаляет задачу из localStorage и переформировывает
    // список всех задач, меняя при этом нумерацию в uid
    deleteTodo(todo) {
      localStorage.removeItem(todo.uid);
      this.lengthTodos = Number(localStorage.getItem('lengthTodos')) - 1;
      let keys = Object.keys(localStorage);
      keys.sort();

      localStorage.setItem('lengthTodos', this.lengthTodos);
      
      for (let i=0; i < this.lengthTodos; i +=1) {
        let receivedTask = JSON.parse(localStorage.getItem(keys[i]));
        localStorage.removeItem(keys[i]);
        receivedTask.uid = i + 1;
        localStorage.setItem(receivedTask.uid, JSON.stringify(receivedTask));
      }
      this.confirmationMessage = 'Задача удалена из списка';
      this.showConfirmation = true;
      this.todos = [];
      this.getTodos();
    },
  },
  components: {
    Confirmation
  },
  created() {
    if (localStorage.getItem('lengthTodos') === null) {
      localStorage.setItem('lengthTodos', 0);
    }
    this.getTodos();
  },
}

</script>

<style scoped>
button#task-add {
  margin-top: 20px;
  margin-bottom: 20px;
}
h1, td {
  text-align: left;
}
.todo-uid {
  text-align: right;
}
</style>
