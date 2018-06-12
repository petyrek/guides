# How to Flow in React

## Prop-types
are a waste of time, use Flow.

## Typing React component with Flow instead

### Simple stateless component
```js
import React from "react"
import type { Animal } from "data/animal/types"

type Props = {
  animals: Animal[],
}

const AnimalList = ({ animals }: Props) => (
  <div>
    <h2>Animal list</h2>
    <ul>
      {animals.map(x => (
        <li>
          {x.id} - {x.name}
        </li>
      ))}
    </ul>
  </div>
)

export default AnimalList
```

### Simple statefull component
```js
import * as React from "react"
import type { Animal } from "data/animal/types"

type Props = {
  animals: Animal[],
}

type State = {
  open: boolean,
}

class AnimalList extends React.Component<Props, State> {
  constructor() {
    super()
    this.state = {
      open: false,
    }
  }

  render() {
    const { animals } = this.props
    const { open } = this.state

    return (
      <div>
        <h2>Animal list</h2>
        {open && (
          <ul>
            {animals.map(x => (
              <li>
                {x.id} - {x.name}
              </li>
            ))}
          </ul>
        )}
      </div>
    )
  }
}

export default AnimalList
```

now our components are type checked when we use it like this
```js
<AnimalList />
```
we get yelled at for not passing `animals` and that's what we want.

But what if we wrap our component in a HoC enhancer?

### Using recompose enhancer
```js
type Props = {
  animals: Animal[],
  loading: boolean,
}

const AnimalList = ({ animals, loading }: Props) => (
  <div>
    <h2>Animal list, {animalsLengthX2}</h2>
    <ul>
      {animals.map(x => (
        <li>
          {x.id} - {x.name}
        </li>
      ))}
    </ul>
  </div>
)

export default branch(props => props.loading, renderComponent(Loader))(AnimalList)
```
Now flow won't complain about this
```js
<AnimalList />
```
because we're passing it onto the enhanced component, not directly to the Component that expects those Props, and so type checking is gone.
Also if our enhancer added some props, we would have to specify that.

We can do better.

### Using recompose enhancer with type inference
```js
const AnimalList = ({ animals, loading, animalsLengthX2 }) => (
  <div>
    <h2>Animal list, {animalsLengthX2}</h2>
    <ul>
      {animals.map(x => (
        <li>
          {x.id} - {x.name}
        </li>
      ))}
    </ul>
  </div>
)

const Loader = () => <div>"loading..."</div>

type Outter = {
  animals: Animal[],
  loading: boolean,
}

const enhancer: HOC<*, Outter> = compose(
  branch(props => props.loading, renderComponent(Loader)),
  withProps(props => ({
    animalsLength: props.animals.length,
  })),
  withProps(props => ({
    animalsLengthX2: props.animalsLength * 2,
  })),
)

export default enhancer(AnimalList)
```
`HOC` is a type import from `recompose` and it takes two type variables, the Inner Component props (*) and Outter Props. We let Flow infer the inner props and it will take care of the rest.

### Combining different kinds of HOCs
```js
const AnimalList = ({ animals, loading, animalsLengthX2 }) => (
  <div>
    <h2>Animal list, {animalsLengthX2}</h2>
    <ul>
      {animals.map(x => (
        <li>
          {x.id} - {x.name}
        </li>
      ))}
    </ul>
  </div>
)

type Props = {
  animals: Animal[],
  loading: boolean,
}

const enhancer: HOC<*, Props> = compose(
  connect(() => ({})),
  withProps(props => ({
    animalsLength: props.animals.length,
  })),
  withProps(props => ({
    animalsLengthX2: props.animalsLength * 2,
  })),
)

export default enhancer(AnimalList)
```
Now we used `connect` HOC from `react-redux` and Flow is now unable to infer the inner type, because `connect` type definition is not compatible.

## The rescue
The easiest way out of this, is to define your own `withConnect` by using `connect` from `react-redux`, like so:
```js
// @flow
import { connect } from "react-redux"
import type { HOC } from "recompose"
import type { StoreState } from "global/store/reducers"

function withConnect<MapperAdd, DispatchAdd, Outter>(
  mapper: (state: StoreState) => MapperAdd,
  dispatch: DispatchAdd,
): HOC<{ ...$Exact<MapperAdd>, ...$Exact<DispatchAdd>, ...$Exact<Outter> },
  Outter,> {
  return connect(mapper, dispatch || {})
}

export default withConnect
```

Now replacing `connect` with `withConnect` will result in type inference working once again!

## Recompose + Statefull component
last but not least, if you need to use statefull component in conjunction with recompose, this is how you'd do it
```js
class AnimalList<P: *> extends React.Component<P> {
  render() {
    const { animals, loading, animalsLengthX2 } = this.props
    return (
      <div>
        <h2>Animal list, {animalsLengthX2}</h2>
        <ul>
          {animals.map(x => (
            <li>
              {x.id} - {x.name}
            </li>
          ))}
        </ul>
      </div>
    )
  }
}

type Outter = {
  animals: Animal[],
  loading: boolean,
}

const enhancer: HOC<*, Outter> = compose(
  withProps(props => ({
    animalsLength: props.animals.length,
  })),
  withProps(props => ({
    animalsLengthX2: props.animalsLength * 2,
  })),
)

export default enhancer(AnimalList)
```
