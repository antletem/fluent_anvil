# Fluent Anvil
This library makes it easy to serve high-quality translated and localized versions of your [Anvil](https://anvil.works/) app. You can make your app appeal to and accessible for everyone. All you need to do is add it as a third party dependency with the token UHLC7WE6TELL25TO . You do not need to download this repository unless you want to contribute (you are welcome to do so) or want to know how it works. 

The library serves as a Python interface to [Fluent](https://projectfluent.org/). It is a localization system developed by Mozilla for natural-sounding translations. In contrast to gettext you can create more nuanced and natural sounding translations. For example, Polish has more plural forms than English or German. Therefore, selecting the right translation requires knowing how many there are of something. With localization systems like gettext, this requires adding additional logic inside your application. With Fluent, this language-specific logic is encapsulated in the translation file and does not impact other translations.

Personally, I think the greatest thing about Fluent apart from the translation quality is that is easier to learn and use than gettext: It uses only one simple text file format (.ftl) and does not require specialized extraction tools. Often, translations are as simple as this:

en_US/main.ftl:
```
close-button = Close
```
de_DE/main.ftl:
```
close-button = Schließen
```

For simple translations, the syntax stays simple. If a translation happens to be more complicated for a language, you only need to add the logic in the translation file for that particular language. You can find out more at [Project Fluent](https://projectfluent.org/).
The translation happens entirely on the client side. Therefore, it works on [Anvil's free plan](https://anvil.works/pricing) as well since there is no need to install a special package.

Please note that this is a personal project with the hope that it may be of use to others as well. I am neither affiliated with [Project Fluent](https://projectfluent.org/) nor [Anvil](https://anvil.works/).

## Quick Guide
In Anvil's assets section, add a directory to place your translations in, ideally you have one subfolder for each locale, e.g.
- localization
     - es_MX
         - main.ftl
     - en_US
         - main.ftl
     - de_DE
         - main.ftl

With Fluent, you can use variables for placeholders or selecting the appropriate translation. In the following example we are going to greet the user. Therefore, we use a variable as a placeholder for the user's name. Assume that the content of es_MX/main.ftl is: 
`hello = Hola { $name }.`

Then, import the fluent singleton object and the Message class in your form (Message is optional but required for the examples):
```py
from fluent_anvil.lib import fluent, Message
```
If you want to know which locale the user prefers, just call
```py
fluent.get_preferred_locales()
```
This will return a list of locales such as `['de-DE']` that the user prefers (this method does not use Fluent but the [get-user-locale](https://www.npmjs.com/package/get-user-locale) package).

Now, you can configure Fluent using the following (we ignore the preferred locale for now):
```py
fluent.configure(["es-MX"], "localization/{locale}/main.ftl")
```
This will tell fluent to use the Mexican Spanish locale. The first parameter is a list of desired locales. Locales are given in the order of preference (most preferable first). This means, Fluent will always try the first locale in the list when trying to find a translation. If a translation is not available for that locale, Fluent will try the others one after another until a suitable translation has been found. The second parameter is a template string indicating where the translation files are stored. The placeholder {locale} is replaced with the desired locale (hyphens converted to underscore, because Anvil does not allow hyphens in directory names). Generally, all methods of the `fluent` Python object accept locales regardless of whether you use hyphens or underscores. Note that you do not have to provide the full URL starting with `./_/theme/`. It will be prepended automatically. If your translation files are stored somewhere else entirely you can explicitly set the prefix by adding it to the end of the parameter list. The template string in the above example is actually the default. So, if you store your translations files in the way outlined above, you can omitt it. In this case, the call becomes simply:
```py
fluent.configure(["es-MX"])
```

Now, you can greet the user:
```py
print(fluent.format("hello", name="John"))
```
Every variable you want to have access to in your .ftl files can be added as a keyword variable. Apart from facts like the user's name this can be used to determine a natural sounding translation. These variables may include the count of something or the time of day. Depending on the type of variable, Fluent will automatically format the value according to the selected locale. For example, these messages:
en_US/main.ftl:
`time-elapsed = Time elapsed: { $duration }s.`
and
de_DE/main.ftl:
`time-elapsed = Vergangene Zeit: { $duration }s.`
After calling a command like
```py
print(fluent.format("time-elapsed", duration=12342423.234 ))
```
the message will show up with locale `en-US` as:
`Time elapsed: ⁨12,342,423.234⁩s.`
While with locale "de_DE" it will show up as:
`Vergangene Zeit: ⁨12.342.423,234⁩s.`
Pay attention to the use of dot and comma which is specific to the respective countries.

You can translate multiple strings at once (that's more efficient than one by one) by wrapping them in Message objects:
```py
print(fl.format(
    Message("hello", name="World"), 
    Message("welcome-back", name="John"),
    ...
))
```
This returns a list of all translations in the same order as the corresponding Message instances. That's nice already. However, my favorite feature is the possibility to write directly to GUI component attributes:
```py
fl.format(
    Message("hello", name="world"), 
    Message(self.label, "text", "hello", name="John"),
)
```
You just provide the component and the name of the attribute you want to write to (similar to Python's `setattr()` function).

You can switch to a different locale on the fly using `set_locale()`. Again, the first list element is the desired locale and the remaining entries in the list are fallback locales in case the translation searched for is not available for the desired locale.
```py
fluent.set_locale(["en-US", "en-GB", "en-AU"])
```

### Bonus Round: Translate your HTML Templates
You can translate your static content as well. Just add the tags `data-l10n-id` for the message id and `data-l10n-args` for context variables (if needed) like this:
```html
<h1 id='welcome' data-l10n-id='hello' data-l10n-args='{"name": "world"}'>Localize me!</h1>
```
If you do not initialize a Fluent instance, you will see "Localize me!". As soon as the Fluent instance is initialized (e.g. with locale es-MX), the text changes to "Hola ⁨world⁩". If Fluent would fail for some reason, the default text (in this case "Localize me!") would be shown.
