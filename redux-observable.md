# redux-observable
When writing redux-based applications, we generally encounter a problem related to handling side effects.

Logout is a great example of this.
Let's suppose we have a `LogoutButton` component.
```js
const LogoutButton = ({ handleLogout }) => (
  <button onClick={handleLogout}>Logout</button>
)

export default compose(
  connect(null, { logoutAction }),
  withRouter,
  withHandlers({
    handleLogout: props => () => {
      props.history.push("/")
      props.logoutAction()
    }
  }),
)(LogoutButton)
```

So we are not only dispatching the `logoutAction`, we're also redirecting the app to `/` route. Logout can probably happen on more ocasions than just this, and so, we would have to repeat this logic all the time.

For this reason, you want to use some kind of redux middleware, that will listen to all the actions happening within the application and act accordingly.

An epic would look like this:
```js
const logoutEpic = action$ =>
  action$
    .ofType("USER/LOGOUT")
    .map(() => push(urls.home))
    .catch(() => Observable.of(push(urls.home)))
```

So now, the only thing we ever have to do is dispatch the action and the epic will take care of the rest.
There is another usecase and that is *handling async*, which redux cannot do by default.

Same logic applies, we only dispatch the action with the intent to `fetch`, and epic will take care of the rest.
More can be read here https://redux-observable.js.org/.
