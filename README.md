npm i --save-exact @types/react-redux@7.1.15 axios@0.21.1 react-redux@7.2.2 redux@4.0.5 redux-thunk@2.3.0

# Redux TS - Redux project

## Think about Design First

 <!-- registry.npmjs.org/-/v1/search?text=react -->

<details>
<summary>Redux design</summary>
<details>
<summary>
Redux Store
</summary>

```js
// Repositories :
// => data : list of repositories from NPM
// => loading : true/false whether we are fetching data
// => error : string, error message if one occurred during fetch
```

![redux components design](design-redux-components.jpg)

</details>

<details>
<summary>Actions</summary>

```js
// Action Creators:
// => searchRepositories(term)

// Actions:
// => SearchRepositories
// => SearchRepositoriesSuccess
// => SearchRepositoriesError

// Actions Types:
// => 'search_repositories'
// => 'search_repositories'
// => 'search_repositories_success'
// => 'search_repositories_error'
```

![redux store design](design-redux-store.jpg)

</details>

<details>
<summary>Redux - Ts Issues to avoid </summary>

```js
// Import can turn messy quickly
// Communacating types to components can be challenging
// Type def files can be over-engineered
```

</details>

</details>

## Path

<!-- src/state/reducers
src/state/actions
src/state/actions-types
src/state/actions-creators -->

<details>
<summary>reducer</summary>

repositoriesReducer.ts

```js
// src/state/reducers
import { ActionType } from "../actions-types";
import { Action } from "../actions";

interface RepositoriesState {
	loading: boolean;
	error: string | null;
	data: string[];
}

const initialState = {
	loading: false,
	error: null,
	data: [],
};

const reducer = (state: RepositoriesState = initialState, action: Action): RepositoriesState => {
	// if (action.type === "search_repositories_success") {
	// 	// 100 % certainty that 'action' satisfies the
	// 	// SearchRepositoriesSuccessAction interface
	// 	action.payload;
	// }

	// switch works as well as if else statement
	switch (action.type) {
		case ActionType.SEARCH_REPOSITORIES:
			return { loading: true, error: null, data: [] };
		case ActionType.SEARCH_REPOSITORIES_SUCCESS:
			return { loading: false, error: null, data: action.payload };
		case ActionType.SEARCH_REPOSITORIES_ERROR:
			return { loading: false, error: action.payload, data: [] };
		default:
			return state;
	}
};

export default reducer;
```

</details>

<details>
<summary>Action</summary>

```js
// src/state/actions
import { ActionType } from "../actions-types";

interface SearchRepositoriesAction {
	type: ActionType.SEARCH_REPOSITORIES;
}
interface SearchRepositoriesSuccessAction {
	type: ActionType.SEARCH_REPOSITORIES_SUCCESS;
	payload: string[];
}
interface SearchRepositoriesErrorAction {
	type: ActionType.SEARCH_REPOSITORIES_ERROR;
	payload: string;
}

export type Action =
	| SearchRepositoriesAction
	| SearchRepositoriesSuccessAction
	| SearchRepositoriesErrorAction;
```

</details>

<details>
<summary>Action-types</summary>

```js
// src/state/actions-types
export enum ActionType {
	SEARCH_REPOSITORIES = "search_repositories",
	SEARCH_REPOSITORIES_SUCCESS = "search_repositories_success",
	SEARCH_REPOSITORIES_ERROR = "search_repositories_error",
}

```

</details>

<details>
<summary>Action-creator</summary>

```js
// src/state/actions-creators
import axios from "axios";
import { Dispatch } from "redux";
import { ActionType } from "../actions-types";
import { Action } from "../actions";

export const searchRepositories = (term: string) => {
	return async (dispatch: Dispatch<Action>) => {
		dispatch({
			type: ActionType.SEARCH_REPOSITORIES,
		});

		try {
			const { data } = await axios.get("https://registry.npmjs.org/-/v1/search", {
				params: {
					text: term,
				},
			});

			const names = data.objects.map((result: any) => {
				return result.package.name;
			});

			dispatch({
				type: ActionType.SEARCH_REPOSITORIES_SUCCESS,
				payload: names,
			});
		} catch (err) {
			dispatch({
				type: ActionType.SEARCH_REPOSITORIES_ERROR,
				payload: "err.message",
			});
		}
	};
};
```

</details>

<details>
<summary>store & index</summary>

```js
// src/state/store.ts
import { createStore, applyMiddleware } from "redux";
import thunk from "redux-thunk";
import reducers from "./reducers";

export const store = createStore(reducers, {}, applyMiddleware(thunk));
```

```js
// src/state/index.ts
export * from "./store";
export * as actionCreators from "./action-creators";
export * from "./reducers/";
```

</details>

<details>
<summary>hooks: useActions & useTypedSelector</summary>

useActions

```js
import { useDispatch } from "react-redux";
import { bindActionCreators } from "redux";
import { actionCreators } from "../state";

export const useActions = () => {
	const dispatch = useDispatch();
	return bindActionCreators(actionCreators, dispatch);
	// {searchRepositories: dispatch(searchRepositories)}
};
```

useTypedSelector

```js
import { useSelector, TypedUseSelectorHook } from "react-redux";
import { RootState } from "../state";

export const useTypedSelector: TypedUseSelectorHook<RootState> = useSelector;
// allow redux to get <RootState> type, when using useTypedSelector
```

</details>

<details>
<summary>Consume & dispatch state</summary>

components/RepositoriesList.tsx

```js
import { useState } from "react";
import { useActions } from "../hooks/useActions";
import { useTypedSelector } from "../hooks/useTypedSelector";

const RepositoriesList: React.FC = () => {
	const [term, setTerm] = useState("");
	const { searchRepositories } = useActions();
	const { data, error, loading } = useTypedSelector((state) => state.repositories);

	const onSubmit = (event: React.FormEvent<HTMLFormElement>) => {
		event.preventDefault();
		searchRepositories(term);
	};

	return (
		<div>
			<form onSubmit={onSubmit}>
				<input value={term} onChange={(e) => setTerm(e.target.value)} />
				<button>Search</button>
			</form>
			{error && <h3>{error}</h3>}
			{loading && <h3>loading...</h3>}
			{!error && !loading && data.map((name) => <div key={name}>{name}</div>)}
		</div>
	);
};

export default RepositoriesList;
```

</details>
