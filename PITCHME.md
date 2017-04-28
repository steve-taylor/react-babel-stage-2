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

This doesn't bleed unnecessary props into the child, but it is **verbose**.

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

### Object rest spread option (stage-2 preset)

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

### Object rest spread option (stage-2 preset)

This is even better, as it destructures all the omitted props into the local
scope so we don't have to prefix them with `this.props`.

However, this can pollute our local scope with unnecessary identifiers.

In reality, it is often useful in React to destructure props in the render
method, so this is usually not a problem.

---

## Functional components

---

```js
class MyComponent extends React.Component {
    render() {
        return (
            <div>
                {this.props.children}
            </div>
        )
    }
}
```

---

```js
const MyComponent = ({children}) => (
    <div>
        {children}
    </div>
);
```
---

### Example

---

```js
import React from 'react';
import PropTypes from 'prop-types';
import noop from 'lodash/noop';
import Fsvg from '@fsa/fsui/src/components/fsvg/fsvg';

export default class RemoteRecord extends React.PureComponent {
    render() {
        return (
            <li className="fiso-sc-card-remote-record">
                <button
                    aria-label={this.props.ariaLabel}
                    onClick={this.props.onClick}
                    onFocus={this.props.onLinkFocus}
                    onBlur={this.props.onLinkBlur}>
                    {this.props.label}
                </button>
                <Fsvg name="next-1" />
            </li>
        );
    }
}

RemoteRecord.defaultProps = {
    label:      'Remote Record',
    ariaLabel:  'Click to record this on Foxtel',
    onClick:     noop,
    onLinkBlur:  noop,
    onLinkFocus: noop
};

RemoteRecord.propTypes = {
    ariaLabel: PropTypes.string,
    label: PropTypes.string,
    onClick: PropTypes.func,
    onLinkBlur: PropTypes.func,
    onLinkFocus: PropTypes.func
};
```

---

```js
import React from 'react';
import PropTypes from 'prop-types';
import noop from 'lodash/noop';
import Fsvg from '@fsa/fsui/src/components/fsvg/fsvg';

const RemoteRecord = ({
    label = 'Remote Record',
    ariaLabel = 'Click to record this on Foxtel',
    onClick = noop,
    onLinkFocus = noop,
    onLinkBlur = noop
}) => (
    <li className="fiso-sc-card-remote-record">
        <button
            aria-label={ariaLabel}
            onClick={onClick}
            onFocus={onLinkFocus}
            onBlur={onLinkBlur}>
            {label}
        </button>
        <Fsvg name="next-1" />
    </li>
);

RemoteRecord.propTypes = {
    ariaLabel: PropTypes.string,
    label: PropTypes.string,
    onClick: PropTypes.func,
    onLinkBlur: PropTypes.func,
    onLinkFocus: PropTypes.func
};

export default RemoteRecord;
```

---

## Bringing it all together (and some recommendations)

---

### `<DropdownMenuGroup />`

---

```jsx
import React from 'react';
import propTypes from 'prop-types';
import noop from 'lodash/noop';

import {fsuiClass, fsuiClassnames} from '../../js/utils/get-package-version';
import ModalDropdownList from '../modal-dropdown-list/modal-dropdown-list';
import Fsvg from '../fsvg/fsvg';

export default class DropdownMenuGroup extends React.Component {
    constructor(props) {
        super(props);
        this.renderDdl = this.renderDdl.bind(this);
        this.state = {revealed: this.props.startRevealed};
        this.toggleState = this.toggleState.bind(this);
    }

    toggleState() {
        this.setState({revealed: !this.state.revealed});
    }

    renderDdl(group, key) {
        const onNewValue = (newValue) => {
            this.props.onChange({[`${group.name}`]: newValue});
        };

        return (
            <ModalDropdownList
                {...group}
                onItemSelected={onNewValue}
                key={key} />
        );
    }

    renderFlyout() {
        if (!this.state.revealed) {
            return null;
        }

        return (
            <span className={fsuiClass('dropdown-menu-group__flyout')}>
                {this.props.modalDdlConfigs.map(this.renderDdl)}
            </span>
        );
    }

    render() {
        const className = fsuiClassnames(
            'dropdown-menu-group',
            {'dropdown-menu-group--revealed': this.state.revealed}
        );
        const revealedLabel = this.state.revealed ? 'Reveal' : 'Hide';

        return (
            <fieldset className={className} role="dialog">
                <span className={fsuiClass('dropdown-menu-group__hitbox-sizer')}>
                    <label>{this.props.label}</label>
                    <button
                        onClick={this.toggleState}
                        aria-label={`${revealedLabel} dropdown menu`}>
                        <Fsvg name="show-hide" />
                    </button>
                </span>
                {this.renderFlyout()}
            </fieldset>
        );
    }
}

DropdownMenuGroup.defaultProps = {
    label:           'Filter By',
    modalDdlConfigs: [],
    startRevealed:   false,
    onChange: noop
};

DropdownMenuGroup.propTypes = {
    label: propTypes.string,
    modalDdlConfigs: propTypes.arrayOf(
        propTypes.shape(ModalDropdownList.propTypes)
    ),
    startRevealed: propTypes.bool,
    onChange: propTypes.func
};
```

---

### `<DropdownMenuGroup />`

with `stage-2`

---

```jsx
import React from 'react';
import propTypes from 'prop-types';
import noop from 'lodash/noop';

import {fsuiClass, fsuiClassnames} from '../../js/utils/get-package-version';
import ModalDropdownList from '../modal-dropdown-list/modal-dropdown-list';
import Fsvg from '../fsvg/fsvg';

export default class DropdownMenuGroup extends React.PureComponent {
    static propTypes = {
        label: propTypes.string,
        modalDdlConfigs: propTypes.arrayOf(
            propTypes.shape(ModalDropdownList.propTypes)
        ),
        startRevealed: propTypes.bool,
        onChange: propTypes.func
    };

    static defaultProps = {
        label: 'Filter By',
        modalDdlConfigs: [],
        startRevealed: false,
        onChange: noop
    };

    state = {
        revealed: this.props.startRevealed
    }

    toggleState = () => {
        this.setState({
            revealed: !this.state.revealed
        });
    };

    renderDdl = (group, key) => {
        const onNewValue = (newValue) => {
            this.props.onChange({[`${group.name}`]: newValue}); // pass both key and value
        };

        return (
            <ModalDropdownList
                {...group}
                onItemSelected={onNewValue}
                key={key} />
        );
    };

    renderFlyout() {
        if (!this.state.revealed) {
            return null;
        }

        return (
            <span className={fsuiClass('dropdown-menu-group__flyout')}>
                {this.props.modalDdlConfigs.map(this.renderDdl)}
            </span>
        );
    }

    render() {
        const className = fsuiClassnames(
            'dropdown-menu-group',
            {'dropdown-menu-group--revealed': this.state.revealed}
        );
        const revealedLabel = this.state.revealed ? 'Reveal' : 'Hide';

        return (
            <fieldset className={className} role="dialog">
                <span className={fsuiClass('dropdown-menu-group__hitbox-sizer')}>
                    <label>{this.props.label}</label>
                    <button onClick={this.toggleState} aria-label={`${revealedLabel} dropdown menu`}>
                        <Fsvg name="show-hide" />
                    </button>
                </span>
                {this.renderFlyout()}
            </fieldset>
        );
    }
}
```

---

### Recommendations:

* Use `<Flyout />` instead of `{this.renderFlyout()}`
* Don't move conditional rendering to the render function or component

---

```jsx
import React from 'react';
import propTypes from 'prop-types';
import noop from 'lodash/noop';

import {fsuiClass, fsuiClassnames} from '../../js/utils/get-package-version';
import ModalDropdownList from '../modal-dropdown-list/modal-dropdown-list';
import Fsvg from '../fsvg/fsvg';

const Flyout = ({
    modalDdlConfigs = [],
    onChange = noop
}) => (
    <span className={fsuiClass('dropdown-menu-group__flyout')}>
        {modalDdlConfigs.map((group, index) => (
            <ModalDropdownList
                {...group}
                key={index}
                onItemSelected={(newValue) => {
                    onChange({[`${group.name}`]: newValue}); // pass both key and value
                }} />
        ))}
    </span>
);


export default class DropdownMenuGroup extends React.PureComponent {
    static propTypes = {
        label: propTypes.string,
        modalDdlConfigs: propTypes.arrayOf(
            propTypes.shape(ModalDropdownList.propTypes)
        ),
        startRevealed: propTypes.bool,
        onChange: propTypes.func
    };

    static defaultProps = {
        label: 'Filter By',
        modalDdlConfigs: [],
        startRevealed: false,
        onChange: noop
    };

    state = {
        revealed: this.props.startRevealed
    };

    toggleState = () => {
        this.setState({
            revealed: !this.state.revealed
        });
    };

    render() {
        const className = fsuiClassnames(
            'dropdown-menu-group',
            {'dropdown-menu-group--revealed': this.state.revealed}
        );
        const revealedLabel = this.state.revealed ? 'Reveal' : 'Hide';

        return (
            <fieldset className={className} role="dialog">
                <span className={fsuiClass('dropdown-menu-group__hitbox-sizer')}>
                    <label>{this.props.label}</label>
                    <button onClick={this.toggleState} aria-label={`${revealedLabel} dropdown menu`}>
                        <Fsvg name="show-hide" />
                    </button>
                </span>
                {revealed && (
                    <Flyout
                        modalDdlConfigs={this.props.modalDdlConfigs}
                        onChange={this.props.onChange} />
                )}
            </fieldset>
        );
    }
}
```

---

### Recommendation:

Use functional components to separate view from logic

---

```jsx
import React from 'react';
import propTypes from 'prop-types';
import noop from 'lodash/noop';

import {fsuiClass, fsuiClassnames} from '../../js/utils/get-package-version';
import ModalDropdownList from '../modal-dropdown-list/modal-dropdown-list';
import Fsvg from '../fsvg/fsvg';

const Flyout = ({
    modalDdlConfigs = [],
    onChange = noop
}) => (
    <span className={fsuiClass('dropdown-menu-group__flyout')}>
        {modalDdlConfigs.map((group, index) => (
            <ModalDropdownList
                {...group}
                key={index}
                onItemSelected={(newValue) => {
                    onChange({[`${group.name}`]: newValue}); // pass both key and value
                }} />
        ))}
    </span>
);


const DropdownMenuGroupView = ({
    revealed,
    onClickToggleReveal,
    label = 'Filter By',
    modalDdlConfigs,
    onChange
}) => (
    <fieldset
        className={fsuiClassnames(
            'dropdown-menu-group',
            {'dropdown-menu-group--revealed': revealed}
        )}
        role="dialog">

        <span className={fsuiClass('dropdown-menu-group__hitbox-sizer')}>
            <label>
                {label}
            </label>
            <button
                onClick={onClickToggleReveal}
                aria-label={`${revealed ? 'Reveal' : 'Hide'} dropdown menu`}>

                <Fsvg name="show-hide" />
            </button>
        </span>

        {revealed && (
            <Flyout {...{modalDdlConfigs, onChange}} />
        )}
    </fieldset>
);


export default class DropdownMenuGroup extends React.PureComponent {
    static propTypes = {
        label: propTypes.string,
        modalDdlConfigs: propTypes.arrayOf(
            propTypes.shape(ModalDropdownList.propTypes)
        ),
        startRevealed: propTypes.bool,
        onChange: propTypes.func
    };

    state = {
        revealed: this.props.startRevealed
    }

    onClickToggleReveal = () => {
        this.setState({
            revealed: !this.state.revealed
        });
    };

    render() {
        return (
            <DropdownMenuGroupView
                {...this.props}
                {...this.state}
                onClickToggleReveal={onClickToggleReveal} />
        );
    }
}
```

---

## Thanks!
