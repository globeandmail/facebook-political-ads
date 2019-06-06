# Internationalization and localization

The FBPAC system — user-facing extension, backend and classifiers — are all built to enable internationalization. Here are the steps you'd need to take to make FBPAC work in your country.

### Translate the extension

Translations for the extension are stored in `https://github.com/globeandmail/fbpac-extension/tree/master/_locales/${locale}/messages.json`. Copy another’s language’s message into a folder for your locale, then translate just the “message” key in `messages.json`.
A locale is a ISO 639-1 language code (e.g. en, de) with an optional ISO 3166-1 Alpha-2 country suffix (e.g. `de_CH`). The translation shown to the user is determined by their browser settings.
 Finally, import your locale and add it to the list of available locales in `https://github.com/globeandmail/fbpac-extension/tree/master/src/locales.js`.
Now, test it. There’s a command in the extension’s `https://github.com/globeandmail/fbpac-extension/tree/master/package.json` that shows how to open a test version of Firefox with a different language’s settings. (But you may need a “language pack” for this command to actually work!)
Add the language and the country to the lists in `https://github.com/globeandmail/fbpac-extension/tree/master/src/i18n.js.` This selection determines the locale associated with ads submitted to the project. Users can select from all known languages and countries while onboarding; this file is what determines which languages and countries are highlighted.
In `https://github.com/globeandmail/fbpac-extension/tree/master/src/i18n.js` a list of active language and country codes can be defined. Active ones get prioritised in the UI.
The FBPAC extension detects advertisements on Facebook using the "Sponsored" tag that appears on each ad. But, this "Sponsored" tag is translated, e.g. to "Commandité" or "Sponsorisé". Find the list of translations around line 585 of extension/src/parser.js and, if your language’s translation not present, add it. (You can easily change your Facebook language to see the UI in whatever target language you’re looking at.)
Note that Facebook inserts the first letter of the Sponsored tag into the middle of the word (e.g. SpSonSsoSred or CoCmmCandCité), presumably as a way to thwart adblockers, and hides the extra letter with CSS. You’ll want to write in the translation of Sponsored “correctly”, without Facebook’s extra junk.

##### Locale-specific styles in extension popup

You can customize, for example font sizes, with `[lang]` and `[data-locale]` CSS selectors:

```css
[lang=de] .toggle {
  font-size: 0.78rem;
}
[data-locale=de_CH] .toggle {
  font-size: 0.78rem;
}
```


### Adding a new model in a target locale

To distinguish political and non-political ads, we need to train a model that’s language and country specific. A model in the correct language but from a different country will probably work tolerably, but a country-specific model will be useful for, say, knowing that “Ford” in Canada is a major politician’s name, but in the U.S. it’s just a car company.

Go to the `https://github.com/globeandmail/fbpac-classifier/` repo. Copy the `data/en-IE` folder to the `data/<your locale>` folder. (The `en-IE` data is just blank, that’s why we’re copying it.) Go into your locale’s classifier_config.json file and change the reference to “en-IE” to your locale. In train_and_upload_models.sh, add your locale to the end of the list on the first line.

Once the model trainer runs (it should be set up on a cron, but if it’s not, run it manually), then the classifier should automatically start classifying ads.

This is the bare minimum. We still haven’t really trained the model. If users (in your locale) actively rate ads, the model will eventually (and automatically) learn from their ratings. However, this will make for a frustrating experience for journalists using the admin, since they’ll have to sort through ALL of the ads… so here’s how to pre-seed the model so it works tolerably from the get-go (and then gets better and better through users’ ratings.)

Looking at seeds.json in your locale. It’s JSON that looks like this: `{ "political": [], "not_political": [] }`. Add text from any source that resembles that of political Facebook ads and of non-political Facebook ads as a list of strings into those arrays.

I suspect that the best way to do that is by scraping tweets from political and non-political Twitter accounts. I haven’t written code to do this. (You could write a generic Twitter scraper. You could even pull the political and non-political handles from seeds_config.json, then write the tweet texts to seeds.json.)

A brief historical digression: until spring 2018, we scraped posts from Facebook pages. However, as a side effect of the Cambridge Analytica scandal, Facebook cut off access (for us and for almost everyone). So for a while, we didn’t do anything. I just recently thought of using Twitter. The analogy before was that Facebook ad texts would resemble Facebook posts, which isn’t exact, but close. Now, the analogy would be that Facebook ad texts would resemble Twitter posts, which is probably a little less close, but still close enough to work.

### Targeting parser

This one's hard.

Facebook provides a brief, incomplete explanation of how an ad was targeted to the user who saw it. [targeting_parser.rs](https://github.com/globeandmail/fbpac-backend/tree/master/server/src/targeting_parser.rs) parses that explanation into attributes the computer can understand. It might look something like this:

_One reason you're seeing this ad is that Donald J. Trump added you to a list of people they want to reach on Facebook. They were able to reach you because you're on a customer list collected by Donald J. Trump or its partners, or you've provided them with your information off of Facebook.
There may be other reasons you're seeing this ad, including that Donald J. Trump wants to reach people ages 18 and older who live in the United States. This is information based on your Facebook profile and where you've connected to the internet._

and [targeting_parser.rs](https://github.com/globeandmail/fbpac-backend/tree/master/server/src/targeting_parser.rs) parses that into `Age: 18 and older, MinAge: 18, Region: the United States, List`. This enables you to, for instance, find all ads targeted to a list of people.

These explanations are often presented in your own language. If you want to parse them, you'll have to modify the parser code in fbpac-backend's [targeting_parser.rs](https://github.com/globeandmail/fbpac-backend/tree/master/server/src/targeting_parser.rs). This is hard! Use the English and the German examples as a guide; it'll probably require a lot of trial and error, along with gathering a wide variety of those targeting descriptions. I don't know an easy way to do it. (And our parser isn't _quite_ perfect even in English.)
