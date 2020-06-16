---
description: Documenting how we handle translations in the code
---

# Translations

We use [react-intl](https://github.com/yahoo/react-intl) to manage our translations. They are extracted from the code to `src/lang/${locale}.json` files using the `npm run build:langs` command \(CI will notify you if the translation files are outdated\). **Don't translate the strings directly in the files**, we use [Crowdin](https://crowdin.com/project/opencollective) to manage our translatations.

## Good practices

### Use "select" when a value has a limited number of options

**Example**

```jsx
<FormattedMessage
    id="withSelect"
    defaultMessage="{action, select, delete {Delete this} archive {Archive this} other {Do something with this}}"
    values={{ action: 'delete' }}
/>

// => "Delete this"

<FormattedMessage
    id="withSelect"
    defaultMessage="{action, select, delete {Delete this} archive {Archive this} other {Do something with this}}"
    values={{ action: 'eat' }}
/>

// => "Do something with this"
```

* `defaultMessage` string breakdown:
  * `action` variable name
  * `select` keyword
  * `delete` and `archive` possible values
  * `other` all other values will use this key

**An exception to this rule:** very common enums or the ones with many possible values should be implemented as a separate file listing all values because:

* Re-usability
* A map of translations is easier to read than a long select string with tons of options

See [i18n-member-role](https://github.com/opencollective/opencollective-frontend/blob/6c164f4b683b5b7393242db537a95c0f033b1377/src/lib/i18n-member-role.js) as an example.

### Don't assume word's order stays the same in other languages

The order of the words may change from a language to another. For this reason we must always pass the values to be replaced in `values` so their order can later be changed.

**Example**

```jsx
// Bad
<div>
    <FormattedMessage id="str" defaultMessage="Pending approval from " />
    <Link route={`/${host.slug}`}>{host.name} </Link>
</div>

// Good
<div>
    <FormattedMessage 
        id="str" 
        defaultMessage="Pending approval from {host}" 
        values ={{ 
          host: <Link route={`/${host.slug}`}>{host.name} </Link>
        }}
    />
</div>
```

### Translate links inline

In some parts of the code we translate links like this:

```jsx
// Please don't do that!
<FormattedMessage
  id="ReadTheDocs"
  defaultMessage="Please check our {documentationLink} to learn more!"
  values={{
    documentationLink: (
      <a href="https://docs.opencollective.com">
        <FormattedMessage id="documentation" defaultMessage="documentation" />
      </a>
    ),
  }}
/>
```

This is bad because we're creating two strings and translators loose the context when they translate one. You should do this instead:

```jsx
<FormattedMessage
  id="ReadTheDocs"
  defaultMessage="Please check our <link>documentation</link> to learn more!"
  values={{
    // eslint-disable-next-line react/display-name
    link: msg => <a href="https://docs.opencollective.com">{msg}</a>,
  }}
/>
```

## FormattedMessage

The `FormattedMessage` component is the main way to translate strings. To use it, you just need to add the following import:

```javascript
import { FormattedMessage } from 'react-intl'
```

Then you just add the component with an unique `id` and a `defaultMessage`.

For VSCode users, you can use the following snippet to make your life easier:

```javascript
{
  "Formatted Message (react-intl)": {
    "scope": "javascript",
    "prefix": "formatted-message",
    "body": "<FormattedMessage id=\"$TM_FILENAME_BASE.$0\" defaultMessage=\"$1\"/>",
    "description": "Put the given string in a FormattedMessage"
  }
}

```

## Add a new language

#### Add the language on Crowdin

* Go to [https://crowdin.com/project/opencollective/settings\#translations](https://crowdin.com/project/opencollective/settings#translations), click on `Target languages` pick the language and click `Update` 

![](../.gitbook/assets/image%20%288%29.png)

* Language is ready for translation!

#### Activate the language in the code

To activate a language on the website, we usually wait to have a correct translated ratio \(20-30%\).

You will need to:

* Copy paste the last line in `frontend/scripts/translate.js` \(replace `it` with your locale code\)

```javascript
translate('it', defaultMessages, diff.updated);
```

* Add the locale in `src/components/Footer.js` for the `languages` object

