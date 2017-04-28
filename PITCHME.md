---

# React with Babel stage-2 preset

---

## Class properties

---?gist=5b7e1ed40926ff24459c3deefcf25587

---?gist=351e58764782d29ea778d56e32b554de

---

Pros:

* Move `propTypes` and `defaultProps` into the component class
* Initialise instance variables at the class level
* No more `this.fn = this.fn.bind(this);`
* No need for a constructor in most cases

Cons:

* You might forget to declare event handlers as lambda functions

---

## Passing props to children

We often want to pass *most* of a parent's components into one of its children.
There are numerous approaches

---

### Lazy option

Pass through all the parent's props to its children, even those props that the
child doesn't use.

---

### Lazy option

```js
render() {
    return (
        <div>
            <button onClick={this.props.onClickToggleReveal}>
                {this.props.toggleButtonLabel}
            </button>
            {this.props.reveal && <UserProfile {...this.props} />}
        </div>
    );
}
```

---

### Lazy option

But this bleeds extra props into the children which can cause bugs when the
child component is modified to use props of the same name. Devs will now have
to be mindful of a possible naming collision.

---

### Manual option

Explicitly pass through each prop from the parent to the child.

---

### Manual option

```js
render() {
    return (
        <div>
            <button onClick={this.props.onClickToggleReveal}>
                {this.props.toggleButtonLabel}
            </button>
            {this.props.reveal && (
                <UserProfile
                    photoUrl={this.props.photoUrl}
                    givenName={this.props.givenName}
                    familyName={this.props.familyName}
                    dateOfBirth={this.props.dateOfBirth}
                    emailAddress={this.props.emailAddress}
                    phoneNumber={this.props.phoneNumber} />
            )}
        </div>
    );
}
```

---

### Manual option

But this is quite verbose.

---

### Lodash option

Use Lodash's `omit` function to use all except a specified list of props.

---

### Lodash option

```js
render() {
    const props = omit(
        this.props,
        ['reveal', 'onClickToggleReveal']
    );

    return (
        <div>
            <button onClick={this.props.onClickToggleReveal}>
                {this.props.toggleButtonLabel}
            </button>
            {this.props.reveal && <UserProfile {...props} />}
        </div>
    );
}
```

---

### Lodash option

This is a good approach as it easily prevents bleeding of unnecessary props
into the child.

---

### Object rest spread option (stage-3 preset)

```js
render() {
    const {reveal, onClickToggleReveal, ...props} = this.props;

    return (
        <div>
            <button onClick={onClickToggleReveal}>
                {toggleButtonLabel}
            </button>
            {reveal && <UserProfile {...props} />}
        </div>
    );
}
```

---

### Object rest spread option (stage-3 preset)

This is even better, as it destructures all the omitted props into the local
scope so we don't have to prefix them with `this.props`.

However, this can pollute our local scope with unnecessary identifiers.

In reality, it is often useful in React to destructure props in the render
method, so this is usually not a problem.

---

## Functional components

--

## Babel stage 3 preset
