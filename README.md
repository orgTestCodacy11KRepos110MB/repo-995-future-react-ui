# A Playground to investigate a good third-party React UI Lib Architecture

My goal is to find an architecture which could be common ground a base for quality React UI libraries. A consistent theming/styling API would make the lives of many developers way more comfortable.

HELP/FEEDBACK/YOUR OPINION WANTED! :)

## How to setup & run

```
npm install
cd app
npm install
npm start
```

## Basic Ideas

#### Class based

Component styles should be class based. It's more performant & responsive styling doesn't work with server-side rendering. I myself went down the inline-styles path in the past, but switched back to classes. Your thoughts?

#### react-themeable

Leverage react-themeable. It's a nice way of providing many styling classes to a single component.
I believe establishing this as a convention would benefit the React community due the consistent API over many libs.

#### Self-contained

Make components self contained (no global UI lib dependencies) so you can import them and avoid importing the whole library e.g.

```
// import the whole lib and get Toggle
import { Toggle } from 'ui-lib';

// just import toggle without importing the whole library
import Toggle from 'ui-lib/Toggle';
```

#### Ship without a theme

Ship without a global theme so people don't have to import the styling code. This might not be relevant to company internal or in general libraries that a opinionated about styling.

#### Global theming utility

Provide a simple & handy way to apply a global theme for all the imported components.

## Global Theming

While react-themeable is super useful I believe having a way to set a default styling is a crucial feature for a UI library. Most of the time you will use the default theme specific to your product. This avoids adding a lot of theme props through the whole application.

#### Module export

https://github.com/nikgraf/future-react-ui/tree/master/ui-lib

In this version the defaultTheme is exported through a named module export. On line 9, 10, 11 you can see how the default theme is patched with custom classes. (https://github.com/nikgraf/future-react-ui/blob/master/app/index.js#L8) Since this is not so hand it might be useful to have utility function for each library to apply a theme.

#### Static property

https://github.com/nikgraf/future-react-ui/blob/master/ui-lib/StaticProperty/Hint.js

In this version the defaultTheme is attached to the component itself as static property. Compared to the 'Module export' this is a bit more flexible as you can overwrite the whole `theme` object in one go. See line 13-20 for usage. (https://github.com/nikgraf/future-react-ui/blob/master/app/index.js#L13)

#### Theme Component leveraging Context

https://github.com/nikgraf/future-react-ui/blob/master/ui-lib/Context/Hint.js

In this version we leverage context to build a `<Theme />` component that takes a theme as property and passes it down to all child components via React's context. A theme is still a simple JS object as can be seen on line 25-30. (https://github.com/nikgraf/future-react-ui/blob/master/app/index.js#L22). See how it's used here: https://github.com/nikgraf/future-react-ui/blob/master/app/index.js#L56. On one hand this approach is powerful, because you can apply different themes various nesting levels in the render tree.

```
<Theme theme={baseTheme}>
  <Theme theme={headerTheme}>
    <Menu items=['Home', 'About'] />
  </Theme>
  <Hint isOpen>Basic open hint without styling.</Hint>
</Theme>
```

There is one obvious concern with this approach. There could be name-clashing between libraries that use the same key in the `theme` object. This could be solved by following a namespace convention like prefixing the keys with the npm package name.

#### Factory Pattern

https://github.com/nikgraf/future-react-ui/tree/master/ui-lib/Factory

In this version we assume there is a package which includes a completely unstyled version of the UI kits while there is a second one which takes all the needed components and return themed components buy leveraging a factory function.

```
/**
 * Returns a the provided component as themed component.
 *
 * Note: defaultProps could be useful for default special behavioural in
 * different ui libraries.
 */
export default (Component, theme, defaultProps) => (props) => {
  return <Component {...defaultProps} theme={theme} {...props} />;
};
```

For example these UI

##### Not styled components

- belle-core (unstyled kit)
- elemental-core (unstyled kit)
- react-toolbox-core (unstyled kit)
- react-select (unstyled component)
- react-autocomplete (unstyled component)
- react-modal (unstyled component)

##### Themed

- belle (themed with belle style and based on belle-core)
- belle-flat (themed with a flat theme and based on belle-core)
- elemental (themed with the elemental theme and based on elemental-core)
- react-toolbox (themed with the material theme and based on react-toolbox-core)
- your-product-ui-lib (themed with your company style and based on belle-core[Toggle, Rating] & react-select & react-modal)
- your-friends-product-ui-lib (themed with their company style and based on react-select & react-modal & react-autocomplete)

Usage:
```
import Hint from 'elemental/Hint';

const customTheme = {
  questionMark: 'custom-class-for-question-mark-gold',
  visibleContent: 'custom-class-visible-content',
};

export default (props) => {
  return (
    <div>
      {/* Globally theme component */}
      <Hint />
      {/* Overwriting the theme locally for this case */}
      <Hint theme={theme}/>
    </div>
  );
};
```

## Temporary Conclusion

While the Theme component based idea is pretty powerful the issues make me not like it as a default. For some time I was convinced that Module export or Static property would be one of the winners. Right now I'm pretty convinced the Factory Pattern is the clear winner. It is super flexible and allows people to create their own company UI kits. They can easily provide their own styling as well as set other props as default suited to their needs. Another benefit is that you can easily get started with prototyping by using an already style version (e.g. belle-flat) and replace it later with your custom product style.

If you have some ideas/feedback please reach out to me and let's discuss. (Github Issues might be best, but Twitter, Email, Skype, Hangout works as well)

## License

MIT
