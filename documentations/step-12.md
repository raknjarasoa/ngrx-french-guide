

## NGRX - ENTITY

### Début de la branche step-12

Cette partie est consacré a de l'optimisation de performance, dans le cas ou notre todo-list contient des milliers de todos on aurez une baisse des performance car car sur chaque action on réalise une itération et si notre todo-list de sois plus un array de todo mais plutôt une **entité** de todo, lors d'un changement on aurai besoin que d'un **object[key]** pour mettre à jour la liste et c'est là que vient [Ngrx/entity](https://github.com/ngrx/platform/blob/master/docs/entity/README.md) ce module permet facilement de prendre en entré un array de créer une entité avec un **adapter** puis de rendre un array lors du **selector**, en plus il fourni des méthodes pour traiter directement avec notre entité comme **AddOne()** ou **AddMany()** .

```bash
npm install @ngrx/entity --save OR yarn add @ngrx/entity --dev
```

```javascript
import { T*todoL-listModule } from '../actions/todo-list.action';
import { TodoListState, Todo  } from '../../models/todo';.reducer.ts*
```javascript
// ... other
import { createEntityAdapter, EntityAdapter, EntityState } from '@ngrx/entity';

export interface TodoListStateEntity extends EntityState<Todo> {
    loading: boolean;
    loaded: boolean;
    selectedTodo: Todo;
    logs: {
        type: string;
        message: string;
    };
}

export const TodoListAdapter: EntityAdapter<Todo> = createEntityAdapter<Todo>({
    sortComparer: false
});

export const initialState: TodoListStateEntity = TodoListAdapter.getInitialState({
    loading: false,
    loaded: false,
    selectedTodo: undefined,
    logs: undefined
});
/*
const initialState: TodoListState = {
    data: [],
    loading: false,
    loaded: false,
    selectedTodo: undefined,
    logs: undefined
};
*/

export function todosReducer(
    state = initialState,
    action: TodoListModule.Actions
): TodoListStateEntity {

  switch (action.type) {

    // GET TODOS
    case TodoListModule.ActionTypes.LOAD_INIT_TODOS:
    export const {
    // select the array of user ids
    selectIds: selectTodosIds,
    // select the dictionary of user entities
    selectEntities: selectTodosEntities,
    // select the array of users
    selectAll: selectTodos,
    // Passe lect the total user count loading a true
        return {
            ...state,
            loading: true
        };

    case TodoListModule.ActionTypes.SUCCESS_INIT_TODOS:
        // Bind state.data avec les todos du server
        // Passe le loaded a true et le loading a false
        return {
            ...TodoListAdapter.addMany(action.payload, state),
            loading: false,
            loaded: true
        };

    // POST TODO
    case TodoListModule.ActionTypes.LOAD_CREATE_TODO:
        // Passe le loading a true
        return {
            ...state,
            loading: true
        };
    selectTotal: selectTotalTodos
  } = TodoListAdapter.getSelectors();


export function todosReducer(
    state = initialState,
    action: TodoListModule.Actions
): TodoListStateEntity {

  switch (action.type) {

    // ... other

    case TodoListModule.ActionTypes.SUCCESS_INIT_TODOS:
        // Bind state.data avec les todos du server
        // Passe le loaded a true et le loading a false
        return {
            ...TodoListAdapter.addMany(action.payload, state),
            loading: false,
            loaded: true
        };

    // ... other

    case TodoListModule.ActionTypes.SUCCESS_CREATE_TODO:
        return {
            ...TodoListAdapter.addOne(action.payload, state),
            loading: false,
            logs: { type: 'SUCCESS', message: 'La todo à été crée avec succès' };

    // SELECT TODO
    case TodoListModule.ActionTypes.SELECT_TODO:
        return {
            ...state,
            selectedTodo: action.payload
        };

    // PATCH TODO

    case TodoListModule.ActionTypes.LOAD_UPDATE_TODO:
        return {
            ...state,
            loading: true }
        };

    // ... other
    
    case TodoListModule.ActionTypes.SUCCESS_UPDATE_TODO:
        const { id, ...changes } = action.payload;
        return {
            ...TodoListAdapter.updateOne({id: id, changes: changes}, state),
            loading: false,
            logs: { type: 'SUCCESS', message: 'La todo à été mise à jour avec succès' }};

        
   // ... other
   // DELETE TODO

    case TodoListModule.ActionTypes.LOAD_DELETE_TODO:
        return {
          };
        };

     // ...state,
            loading: true
        }; other
   
    case TodoListModule.ActionTypes.SUCCESS_DELETE_TODO:
        return {
            ...TodoListAdapter.removeOne(action.payload, state),
            logs: { type: 'SUCCESS', message: 'La todo à été suprimmé avec succès' }
        };

// ... other

```
    case TodoListModule.ActionTypes.ERROR_LOAD_ACTION:
        return {
            ...state,
            loading: false,
            logs: { type: 'ERROR', message: action.payload.message }
        };

    default:
        return state;
    }
}

```
```javascript
// ... other

export interface AppState {
    todos: TodoListStateEntity;
}

// ... other

```

```javascript
import { createSelector } from '@ngrx/store';
import { AppState } from '..';

/export const selectT    }
        };


*store/index.ts*
```javascript
import { TodoListStateEntity } from './reducers/todo-list.reducer';

// ... other

// Changement de l'interface
export interface AppState {
    todos: TodoListStateEntity;
}

// ... other

```
*todo-list.selector.ts*
```javascript
// ... other
import * as fromTodos from '@Reducers/todoL-list.reducer';

export const selectTodoListState$ = (state: AppState) => state.todos;

/*.reducer';

// ... othere  te Ae tte
export const selectTodos$ =
    createSelector(selectTodoListState$, (todos) => todos.data);
*/

export const selectTodoListEntities$ = Converted$ createSelector(
    selectTodoListState$,
    (state) => state.entities
);

export const selectTodoListEntitiesConverted$ = createSelector(
    selectTodoListState$,Entities$,
    (entities) => Object.keys(entities).map(id => entities[parseInt(id, 10)])
 fromTodos.selectTodos);

export const selectTodoSelected$ =
    createSelector(selectTodoListEntitiesState$, (todos) => todos.selectedTodo);

export const selectTodosLoaded$ =
    createSelector(selectTodoListEntitiesState$, (todos) => todos.loaded);

export const selectTodosErrors$ =
    createSelector(selectTodoListEntitiesState$, (todos) => todos.logs);


```
Switch du selector dans le component: 

*all-todos.component.ts*
```javascript
// ... other

import { 
	selectTodoListEntitiesConverted$ 
} from '@Selectors/todo-list.selector';

// ... other

  constructor(
    // ... other
  ) {
    this.todos$ = store
      .pipe(select(selectTodoListEntitiesConverted$));


// ... other

```
Voilà nos todos sont stockées en tant que entité dans notre state.

<!--stackedit_data:
eyJoaXN0b3J5IjpbMTQ5NzU4ODkwNCwxMjA2NjgxNDg1LC0yMD
E4NzA5NzQ0XX0=
-->