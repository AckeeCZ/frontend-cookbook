# 📖 Paging implementation

This is guide of building very easy but powerful paging in your _React_ application app. 

## Prerequisites

* You need to use _Redux_ and _React router_ ([connected router](https://github.com/supasate/connected-react-router) of course) in your app 
* **There is only one paging on the webpage.** If you need more paging on the webpage then you will have to implement it by yourself, but you can still find some parts of the code below useful.
* Installed our [@ackee/chris](https://www.npmjs.com/package/@ackee/chris) package because we will need its [`routeDependencies`](https://github.com/AckeeCZ/chris#routedependenciesconfig-config--component--componentwithroutedependencies) HOC

## Paging module sources

Start with config that defines default values that are used when parameters for paging miss at URL or have invalid values:

```js
// modules/paging/services/config.js
export default {
    defaultValues: {
        page: 1,
        limit: 10,
    },
};
```

Then create selector that parse URL and get paging parameters from it or use the default values if they have invalid values:

```js
// modules/paging/services/selectors.js
import { createSelector } from 'reselect';
import qs from 'query-string';
import { routingSelector } from '@ackee/chris';

import config from './config';

export const pagingSelector = createSelector(
    routingSelector,
    ({ search }) => {
        const query = qs.parse(search);
        const page = Number(query.page);

        return {
            page: !isNaN(page) && page >= 1 ? page : config.defaultValues.page,
            limit: config.defaultValues.limit,
        };
    },
);
```

Create a component to display content paging. It receives current `page`, `limit` (count of records on one page), `totalCount` of records and change page handler function.  
All props are provided by Pagination container (described in next step), except `totalCount` which has to supplied from `Pagination` parent component:

```jsx
// modules/paging/components/Pagination.jsx
import React from 'react';
import PropTypes from 'prop-types';
import Pagination as AntdPagination from 'antd/lib/pagination';

const Pagination = ({ page, totalCount, limit, setPage }) => (
    <AntdPagination current={page} total={totalCount} pageSize={limit} onChange={setPage} />
);

Pagination.propTypes = {
    totalCount: PropTypes.number.isRequired,
    page: PropTypes.number.isRequired,
    limit: PropTypes.number.isRequired,
    setPage: PropTypes.func.isRequired,
};

Pagination.defaultProps = {
    vacations: [],
};

export default Pagination;
```

Create container that use the selector to get paging parameteres from _React router_'s state and pass them to the `Pagination` component. Container is also enhanced with `withRouter` which means that router objects like `history`, `location`, etc. are supplied to props. Both mentioned props are used to in change page handler to set new page into URL's query string:

```js
// modules/paging/containers/Pagination.js
import { compose } from 'redux';
import { connect } from 'react-redux';
import { withRouter } from 'react-router';
import { withProps } from 'recompose';
import qs from 'query-string';

import { propertiesToQuery } from '../../utilities/propertiesToQuery.js';
import { pagingSelector } from '../services/selectors';
import Pagination from '../components/Pagination';

export default compose(
    withRouter,
    connect(pagingSelector),
    withProps(({ history, location }) => ({
        setPage: page => {
            history.push({ 
                ...location,
                search: qs.stringify({ ...qs.parse(location.search), page })
            });
        },
    })),
)(Pagination);
```

All that your paging module needs to export is just selector and `Pagination` container:

```js
// modules/paging/index.js
export { default as Pagination } from './containers/Pagination';
export * from './services/selectors';
```

That's all! No saga or reducer needed, just selector and connected component..

### Usage

Now let's wire it up! 

Take the `Pagination` from module and supply records total count to it:

#### Component
```jsx
// components/UsersPage.js
import { React } from 'react';
import UsersList from './containers/UsersList';
import { Pagination } from '../modules/paging';

const UsersPage = ({ users, usersTotalCount }) => (
    <React.Fragment>
        <UsersList />
        <Pagination totalCount={usersTotalCount} />
    </React.Fragment>
);

export default UsersPage;
```

#### Container

Leverage the `routeDependencies` (mentioned in [prerequisites](#prerequisites)) enhancer for requesting users. Since the enhancer implementation does refetch every time any part (including query string) of url change, it's ensured that when you change the page, new users list is loaded:

```js
// containers/UsersList.js
import { compose, bindActionCreators } from 'redux';
import { connect } from 'react-redux';
import { routeDependencies } from '@ackee/chris';
import { requestUsers, clearUsers } from '../services/usersActions';
import { usersDataSelector, fetchingUsersSelector } from '../services/selectors/usersSelectors';

export default compose(
    withRouter, // we need this to enable rerender when url query string change
    connect(
        state => ({
            Users: usersDataSelector(state),
            fetchingUsers: fetchingUsersSelector(state),
        }),
        dispatch =>
            bindActionCreators(
                {
                    fetch: requestUsers,
                    clear: clearUsers,
                },
                dispatch,
            ),
    ),
    routeDependencies(),
)
```

#### Params selector

It's also good (but not necessary) to create a selector that select paging parameters and transform them to the form that is required for our API call:


```js
// selectors/index.js
import { createSelector } from 'reselect';
import { pagingSelector } from '../modules/paging';

export const queryStringParamsSelector = createSelector(
    pagingSelector,
    ({ page, limit }) => {
        return {
            offset: (page - 1) * limit,
            limit,
        };
    },
);
```

> Note: Names of API paging parameters (`offset` and `limit` in our example) may differ across projects so feel free to adjust them as your API requires.

#### Saga

And finally in the saga (if you uses `redux-saga`) or elsewhere, use the selector from above to get params for API call:

```js
// sagas/usersSaga.js
import { put, select, takeLatest } from 'redux-saga/effects';

import { requestUsersSucceeded, requestUsersFailed, requestUsersCompleted } from '../actions/usersActions';
import { queryStringParamsSelector } from '../selectors';
import actionTypes from '../actionTypes';
import { api } from '../config';

export default function*() {
    yield takeLatest([actionTypes.REQUEST_USERS], function*() {
        try {
            const options = { 
                resolveWithFullResponse: true,
                qs: yield select(queryStringParamsSelector)
            };

            const response = yield authApi.get(api.users, options);
            const users = response.body;
            const totalCount = Number(response.headers.get('X-Total-Count'));

            yield put(requestUsersSucceeded(Users, totalCount));
        } catch (e) {
            yield put(requestUsersFailed(e));
        } finally {
            yield put(requestUsersCompleted());
        }
    });
}
```

## Conclusion

Here we are, that's all you need to quickly implement pagination by using _Redux_, _React router_ and `routeDependencies` enhancer. Remember that implementation is slightly simplified for purpose of this recipe so if you fell it could be programmed better, don't hesitate and adjust it as you need.

### Extending with search

The paging solution is easy extendable, eg. if you also need fulltext search for the list, you only have to make few changes:

Create your `UsersSearch` component, notice how it is similar to [`Pagination`](#container) container. If search input is empty set the value to `undefined` which causes it's removed from the query string as it's useless there:

```js
// containers/UsersSearch.js
import { compose, withProps } from 'recompose';
import { withRouter } from 'react-router';
import {  } from 'query-string';

const UsersSearch = ({ onSearch }) => (
    <input type="text" onChange={e => onSearch(e.target.value)} />
);

export default compose(
    withRouter,
    withProps(({ history, location }) => ({
        onSearch: query => {
            query = query === '' ? undefined : query;
            history.push({
                ...location,
                search: qs.stringify({ ...qs.parse(location.search), page: 1, query })
            });
        },
    })),
)(UsersSearch);
```

> Note 1: Normally container and view component should been divided into two files, but for our example, we keep them together.

> Note 2: In real application the input `onChange` handler should been debounced to prevent api call on every character user type.

Add new selector for getting search expression from url query string, and modify [`queryStringParamsSelector`](#params-selector) to return also this expression:

```js
// selectors/index.js
import { createSelector } from 'reselect';
import { pagingSelector } from '../modules/paging';

export const filterQuerySelector = createSelector(
    routingSelector,
    ({ search }) => qs.parse(search).query,
);

export const queryStringParamsSelector = createSelector(
    filterQuerySelector,
    pagingSelector,
    (query, { page, pageLimit }) => {
        const queryStringParams = {
            offset: (page - 1) * pageLimit,
            limit: pageLimit,
        };

        if (query) {
            queryStringParams.q = query;
        }
    },
);
```

> Note: Same as for `offset` and `limit` ([mentioned earlier](#params-selector)) the `q` is the query parameter name of our API. Be sure your API uses the same names otherwise change it to make it works.

And that's all, you got an extra searching feature almost for free 🎉