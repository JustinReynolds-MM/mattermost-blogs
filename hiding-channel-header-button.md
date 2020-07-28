# What's the problem?

Channel header buttons are <something about their convenience> but once registered, they show up in all channels. Depending on the use-case, we may not want to show them all the time and in all the channels. The's a neat little trick we can use to conditionally show or hide the button.

![image](channel-header-button.png)

# Button Component

We'll consider a bare minimum channel hedaer button component. Nothing fancy, with a simple render function retunring a SVG image wrapped inside a span.

```JSX
class ChannelHeaderButtonIcon extends React.Component {
    constructor(props) {
        super(props);
        this.state = this.getInitialState();
    }
    
    getInitialState = () => {
        return {
            // ... some state params
        };
    };
    
    render() {
        return (
            <span
                className={'raven-icon'}
                dangerouslySetInnerHTML={{
                    __html: logo,
                }}
            />
        );
    }
}
```

# Adding Refs

The way we'll be hiding the channel header button is by adding or removing a `hidden` class from button component's wrapper. By wrapper we don;'t mean the `span` wrapper we have in out component, but rather the plugin channel header button wrapper Mattermost webapp adds to all all plugin channel header buttons.

To access this wrapper inside out React component we'll add refs to our component and navigate our way above in DOM hierarchy.


## Parent ref placeholder in state

Add parent ref palceholder in component's state-

```JSX
getInitialState = () => {
  return {
      // ... some state params
      parent: undefined,
  };
};
```

## Get parent element ref

Add ref to wrapper `span` in render method-

```JSX
render() {
    return (
        <span
            ref={this.handleRef}
            className={'raven-icon'}
            dangerouslySetInnerHTML={{
                __html: logo,
            }}
        />
    );
}
```

Get parent ref from `span`-

```JSX
handleRef = (ref) => {
    if (ref) {
        this.setState({
            parent: ref.parentNode,
        });
    }
}
```

`ref.parentNode` gives us the parent element.

## Show or hide the button

Now we can use any condition we need and simply add or remove `hidden` class to the parent to show or hide it. We'll do this in `render()` method.

```JSX
render() {
    if (this.state.parent) {
        if (<some condition>) {
            // if true, hide the element
            this.state.parent.classList.remove('hidden');
        } else {
            // else display it
            this.state.parent.classList.add('hidden');
        }
    }

    return (
        <span
            ref={this.handleRef}
            className={'raven-icon'}
            dangerouslySetInnerHTML={{
                __html: logo,
            }}
        />
    );
}
```

To toggle the button in specific channels we can receive current channel ID here and include it in our condition. Mattermost already include current channel in the action handler for channel hedaer buttons. The channel ID can be obtained from this.

# Handling the case of plugin dropdown menu

When more than five channel header buttons are registered, Mattermost combines them all into a single dropdown menu. In this case the parent node we're interested in changes.

## Finding the right parent node.

We can navigate the DOM and check for presence of certain classes in one of the parent nodes to detect if we're in dropdown mode. Based on this we have two different parent nodes we want to alter.

```JSX
getIconParentToHide = () => {
    if (this.isChannelHeaderButtonInDropdown()) {
        return this.state.parent.parentNode.parentNode;
    }
    return this.state.parent;
}


isChannelHeaderButtonInDropdown = () => {
    const classList = this.state.parent.parentNode.parentNode.parentNode.parentNode.classList;
    return classList.contains('dropdown') && classList.contains('btn-group');
}
```

Now instead of changing classes on `this.state.parent`, we do it on the DOM node returned by `getIconParentToHide()`-

```JSX
render() {
    if (this.state.parent) {
        const targetParent = this.getIconParentToHide();
        if (<some condition>) {
            // if true, hide the element
            targetParent.classList.remove('hidden');
        } else {
            // else display it
            targetParent.classList.add('hidden');
        }
    }

    return (
        <span
            ref={this.handleRef}
            className={'raven-icon'}
            dangerouslySetInnerHTML={{
                __html: logo,
            }}
        />
    );
}
```

