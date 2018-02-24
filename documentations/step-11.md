## ERROR LOG

### Début de la branche step-11

Voilà le **mvp** de la todoList est terminé, sur la suite du tutoriel on va voir comment optimiser notre code et le 1er point c'est concernant les actions d'erreurs qui dans le reducer sont répété in fine par action, on pourrait les fusionné pour n'avoir que un state d'erreur todo-list :

*todo-list.action.ts*
```javascript
// ... Other
export namespace TodoListModule {

    export enum ActionTypes {
	    // ... Other
        // ERROR_CREATE_TODO = '[todoList] Error Create Todo',
        // ERROR_UPDATE_TODO = '[todoList] Error Update Todo',
        // ERROR_DELETE_TODO = '[todoList] Error Delete Todo',
        // ERROR_INIT_TODOS = '[todoList] Error Init Todos',
        // Error request Todos
        ERROR_LOAD_ACTION = '[todoList] Error Load Action'
    }

	// ... Other
    /*
    export class ErrorInitTodos {
        readonly type = ActionTypes.ERROR_INIT_TODOS;
    }
    // ... Other
    /*
    export class ErrorCreateTodo {
        readonly type = ActionTypes.ERROR_CREATE_TODO;
    }
    */
	// ... Other
    /*
    export class UpdateTodo {
        readonly type = ActionTypes.UPDATE_TODO;
        constructor(public payload: Todo) {}
    }
    */
	// ... Other
    /*
    export class ErrorUpdateTodo {
        readonly type = ActionTypes.ERROR_UPDATE_TODO;
    }
    */

    // ... Other
    
    /*
    export class ErrorDeleteTodo {
        readonly type = ActionTypes.ERROR_DELETE_TODO;
    }
    */

    // ERROR ACTIONS

    export class ErrorLoadAction {
        readonly type = ActionTypes.ERROR_LOAD_ACTION;
    }

    export type Actions = LoadInitTodos
        // ... other
        // | ErrorInitTodos
        // | ErrorCreateTodo
        // | ErrorUpdateTodo
        // | ErrorDeleteTodo

}

```
*todo-list.reducer.ts*
```javascript
// ... other
  switch (action.type) {

    // ... other
    /*
    case TodoListModule.ActionTypes.ERROR_INIT_TODOS:
        // Error rend le loading a false
        return {
            ...state,
            loading: false
        };*/

    // ... other
    /*
    case TodoListModule.ActionTypes.ERROR_CREATE_TODO:
        // Passe le loading a true
        return {
            ...state,
            loading: false
        };*/

    // ... other
    /*
    case TodoListModule.ActionTypes.UPDATE_TODO:
        return {
            ...state,
            data: state.data
                .map(todo => action.payload.id === todo.id ? action.payload : todo)
        };
        */

    // ... other
    /*
    case TodoListModule.ActionTypes.ERROR_UPDATE_TODO:
        return {
            ...state,
            loading: false
        };*/


    // ... other
    /*
    case TodoListModule.ActionTypes.ERROR_DELETE_TODO:
        return {
            ...state,
            loading: false
        };*/

    case TodoListModule.ActionTypes.ERROR_LOAD_ACTION:
        return {
            ...state,
            loading: false
        };

    // ... other
}

```
*todo-list.effect.ts*
```javascript
// ... other
@Injectable()
export class TodoListEffects {
  // Listen les actions passées dans le Store
    @Effect() LoadTodos$: Observable<TodoListModule.Actions> = this.actions$
      .pipe(
          // ... other
          catchError(() => of(new TodoListModule.ErrorLoadAction()))
      );

    @Effect() LoadCreateTodo$: Observable<TodoListModule.Actions> = this.actions$
      .pipe(
          // ... other
          catchError(() => of(new TodoListModule.ErrorLoadAction()))
      );

    @Effect() LoadDeleteTodo$: Observable<TodoListModule.Actions> = this.actions$
      .pipe(
          // ... other
          catchError(() => of(new TodoListModule.ErrorLoadAction()))
      );

    @Effect() LoadUpdateTodo$: Observable<TodoListModule.Actions> = this.actions$
      .pipe(
      // ... other
          catchError(() => of(new TodoListModule.ErrorLoadAction()))
      );
  // ... other
}

```
Voilà on a purger un peu de code inutile mais on peu créer un système de logs en cas d'erreur, il faut savoir que dans le catchError, il peut prendre en argument l'erreur :

```javascript
catchError((err) => of(new TodoListModule.ErrorLoadAction()))
```
Donc on peut récupérer cette erreur pour afficher le message dans un la view avec avec un **toastr**

*todo-list.action.ts*
```javascript
import { HttpErrorResponse } from '@angular/common/http';
// ... other
export class ErrorLoadAction {
    readonly type = ActionTypes.ERROR_LOAD_ACTION;
    constructor(public payload: HttpErrorResponse) {}
}
```
et passer l'erreur sur chaque actions dans les effects :

*todo-list.effect.ts*
```javascript
// ... other
catchError((err) => of(new TodoListModule.ErrorLoadAction(err)))
```
On ajoute une autre propriété qui permettra de garder les erreurs: 

*models/todo.ts*
```javascript
export interface TodoListState {
    // ... other
    logs: {
        type: string;
        message: string;
    };
}
```
Ajoutez cette propriété dans le reducer: 

*todo-list.reducer.ts*
```javascript
const initialState: TodoListState = {
    // ... other
    logs: undefined
};

case TodoListModule.ActionTypes.SUCCESS_DELETE_TODO:
        return {
            ...state,
            data : state.data.filter(todo => todo.id !== action.payload),
            logs: { type: 'SUCCESS', message: 'La todo à été suprimmé avec succès' }
        };

case TodoListModule.ActionTypes.SUCCESS_UPDATE_TODO:
        return {
            ...state,
            loading: false,
            logs: { type: 'SUCCESS', message: 'La todo à été mise à jour avec succès' },
            data: state.data
                .map(todo => action.payload.id === todo.id ? action.payload : todo)
        };

case TodoListModule.ActionTypes.SUCCESS_CREATE_TODO:
     return {
         ...state,
         logs: { type: 'SUCCESS', message: 'La todo à été crée avec succès' },
         data: [
             ...state.data,
             action.payload
         ]
     };
        
case TodoListModule.ActionTypes.ERROR_LOAD_ACTION:
    return {
        ...state,
        loading: false,
        logs: { type: 'ERROR', message: action.payload.message },
    };
```
On aura besoin d'un selector pour le logs: 

*todo-list.selector.ts*
```javascript
export const selectTodosErrors$ =
    createSelector(selectTodoListState$, (todos) => todos.logs);
```
Maintenant on installe le module [ngx-toastr](https://github.com/scttcper/ngx-toastr) et une fois que vous aurez tout bien installer allez dans le **TodoListComponent**

```javascript
import { Component } from '@angular/core';
import { ToastrService } from 'ngx-toastr';
import { AppState } from '@StoreConfig';
import { Store, select } from '@ngrx/store';
import { Observable } from 'rxjs/Observable';
import { selectTodosErrors$ } from '@Selectors/todo-list.selector';
import { tap } from 'rxjs/operators';

@Component({
  selector: 'app-todo-list',
  template: `
	  <header>
		  <nav>
			  <a routerLink="all-todos" routerLinkActive="active">all todos</a>
			  <a routerLink="select-todo" routerLinkActive="active">select todo</a>
		  </nav>
	  </header>
	  <router-outlet></router-outlet>
  `,
  styleUrls: ['./todo-list.component.scss']
})
export class TodoListComponent {

  public todoListErrors$: Observable<any>;

  constructor(
    private toastr: ToastrService,
    private store: Store<AppState>
  ) {
    this.todoListErrors$ = store.pipe(
      select(selectTodosErrors$),
      tap((dialog) => {
        if (!dialog) {
          return;
        }
        if (dialog.type === 'ERROR') {
          this.toastr.error(dialog.message);
        } else {
          this.toastr.success(dialog.message);
        }
        console.log(dialog);
      })
    );
    this.todoListErrors$.subscribe();
  }

}

```

### Fin de la branche step-11
<!--stackedit_data:
eyJoaXN0b3J5IjpbLTE4NjE3OTU4ODEsLTk1Mjk1ODg4XX0=
-->