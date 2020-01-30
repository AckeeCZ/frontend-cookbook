# Firebase authentication
If you need fast and simple authentication for your web app, Firebase is the right tool for you. You can support custom email and password authentication and/or services like Google, Facebook, etc.

## Table of contantes

* [Basic React/Redux app authentication](#basic-reactredux-app-authentication)
* [Social authentication providers](#social-authentication-providers)
    * [UI part](#ui-part)
    * [Business logic](#business-logic)
    * [Helper functions](#helper-functions)

## Basic React/Redux app authentication
Everything about web authentication is well documented in [Firebase docs](https://firebase.google.com/docs/auth/web/start).

### Tips
 * Install firebase through npm: `npm i --save firebase`
 * Initialize Firebase among with store (typically in `createStore.js` file)
 * Save info about authenticated user in Redux store
 * Wrap your `App` component with *higher order component* observing user's authentication state
 * Use [Redux Saga channels](https://redux-saga.js.org/docs/advanced/Channels.html) for Firebase auth state observing logic

```js
// Login form submit handler
function* handleLoginForm(action) {
    ...
    yield firebase.auth().signInWithEmailAndPassword(email, password);
    ...
}

// Factory function returning channel with Firebase auth state observer
function getAuthChannel() {
    return eventChannel(emit => {
        return firebase.auth().onAuthStateChanged(user => emit({ user }));
    });
}

// Auth state observing logic
function* observeUserFirebase() {
    yield put(authActions.firebaseAuthFetching(true));
    const authChannel = yield call(getAuthChannel);

    while (true) {
        const { user } = yield take(authChannel);
        yield put(authActions.setUser(user));
    }
}
 ```

 ## Social authentication providers

This is a brief description of how to authenticate with [Google](https://firebase.google.com/docs/auth/web/google-signin), [Facebook](https://firebase.google.com/docs/auth/web/facebook-login) or [Apple](https://firebase.google.com/docs/auth/web/apple) account in the Redux saga driven app. The flow is controlled by our authentication library [Petrus](https://www.npmjs.com/package/@ackee/petrus) and uses [Firebase auth](https://firebase.google.com/docs/auth/web/start) library.

> It's not a complete copy/paste-in example. There are some minor parts missing or simplified for the sake of keeping the recipe short & readable so don't hesitate to adjust it to your needs 😉

### UI part

For social login components we can use [`react-facebook-login`](https://www.npmjs.com/package/react-facebook-login), [`react-google-login`](https://www.npmjs.com/package/react-google-login) or [`react-apple-login`](https://www.npmjs.com/package/react-apple-login) are pretty well customizable and have suitable interface. But they still require some configuration and flow driving and we can let Firebase do lot of that for us.

```js
const SocialLoginType = {
    GOOGLE: 'google',
    FACEBOOK: 'facebook',
    APPLE: 'apple',
};
```

the `SocialLoginType` enum is used to identify which login type was used by user

```jsx
<button onClick={() => loginWithSocialProvider(SocialLoginType.GOOGLE)}>
    Login with Google
</button>
```

### Business logic

As described at [Petrus's documentation](https://www.npmjs.com/package/@ackee/petrus#usage), we must provide `authenticate`, `refreshTokens` and `getAuthUser` sagas to it's `configure` function.

```js
// import firebase package and also its auth module
import firebase from 'firebase/app';

import 'firebase/auth';

const firebaseApp = firebase.initializeApp(Config.firebase.app);
const firebaseAuth = app.auth();

 function* authenticate(payload) {
    yield firebaseAuth.setPersistence(firebaseAuth.Auth.Persistence.LOCAL);

    const user = yield signInWithSocialProvider(payload);
    const accessToken = yield getAccessToken(user);

    return {
        tokens: {
            accessToken,
            refreshToken: {
                token: null,
            },
        },
        user: getUserData(user),
    };
}

function* refreshTokens({ refreshToken }) {
    const user = yield getUserFromFirebase();
    const accessToken = yield getAccessToken(user);

    return {
        accessToken,
        refreshToken,
    };
}

export default function* getAuthUser({ accessToken }) {
    const user = yield getUserFromFirebase();

    return getUserData(user);
}
```

as you certainly noticed, there is few helpers used in the functions. Each of them will be described separately in next section.

Last step to wire components up with Petrus, is to catch action dispatched by`loginWithSocialProvider`  and tell Petrus to begin logging process. Payload of the action is an object with social login type [ dispatched from Login component](#ui-part).

```js
function* handleSocialLogin(action) {
    yield put(Petrus.loginRequest(action.payload));
}

function handleLoginFailed({ error }) {
    Log.error('Error encountered', error);
}

export default function*() {
    yield all([
        yield takeEvery(types.LOGIN_WITH_SOCIAL_PROVIDER, handleSocialLogin),
        yield takeEvery(types.LOGIN_FAILURE, handleLoginFailed),
    ]);
}
```


### Helper functions

**`getAccessToken`**

Main purpose of this helper is to return token along with its expiration period. This is important since otherwise Petrus doesn't recognize expired token and user would be stuck in state where he is logged in, but his access token wouldn't work for API requests.

`user`is [`firebase.User`](https://firebase.google.com/docs/reference/js/firebase.User) object containing information about logged user.  

```js
import jwtDecode from 'jwt-decode';

function* getAccessToken(user) {
    const token = yield user.getIdToken(true);
    const { exp } = jwtDecode(token);
    const expiration = new Date(exp * 1000).toISOString();

    return {
        token,
        expiration,
    };
}
```

**`getUserData`**

Common helper to extract basic user information from the data provided by Google/Facebook stored in object [`providerData`](https://firebase.google.com/docs/reference/js/firebase.User.html#providerdata). Can be extended as needed, but often this set of information is enough.

```js
import { get } from 'lodash';

const getUserData = user => {
    const data = get(user, 'providerData[0]', user);

    return {
        name: data.displayName,
        email: data.email,
        photoURL: data.photoURL,
    };
};
```

**`getUserFromFirebase`**

When called, it will return [`firebase.User`](https://firebase.google.com/docs/reference/js/firebase.User) as soon and it is available.  

```js
function authStateChanged() {
    return new Promise(resolve => {
        firebaseApp.auth.onAuthStateChanged(user => {
            resolve(user);
        });
    });
}

export default async function getUserFromFirebase() {
    const { currentUser } = firebaseApp.auth;

    if (currentUser) {
        return currentUser;
    }

    const user = await authStateChanged();

    return user;
}
```

**`signInWithSocialProvider`**

This is crucial part of the flow. Once we call `signInWithPopup`, firebase take care of authenticating using one of defined social login providers (includes opening new window) and instantly return just authenticated user. We are using sign in with opening new window, but there are also `signInWithRedirect` and [others available](https://firebase.google.com/docs/reference/js/firebase.auth.Auth#methods).


```js
export default async function signInWithSocialProvider(loginData = {}) {
    const { loginType } = loginData;

    let provider;
    let scopes;

    switch (loginType) {
        case LoginType.GOOGLE:
            provider = new firebase.auth.GoogleAuthProvider();
            scopes = ['profile', 'email'];
            break;
        case LoginType.FACEBOOK:
            provider = new firebase.auth.FacebookAuthProvider();
            scopes = ['public_profile', 'email'];
            break;
        case LoginType.APPLE:
            provider = new firebase.auth.OAuthProvider('apple.com');
            scopes = ['name', 'email'];
            break;
        default:
            Log.warn(`Unknown login type ${loginType}`);
            return null;
    }

    for (let scope of scopes) {
        provider.addScope(scope);
    }

    const { user } = await firebaseApp.auth.signInWithPopup(provider);

    return user;
}
```

---

Now just configure Petrus the standard way, create needed actions, use code described in the recipe and it should work!  
All the code is extracted from a real project, just a bit simplified and deprived of project specific code.