# Facebook Political Ad Collector

This is the source code behind our project to collect political ads on Facebook. Built versions are available for [Firefox](https://addons.mozilla.org/en-US/firefox/addon/facebook-ad-collector/) and [Chrome](https://chrome.google.com/webstore/detail/facebook-political-ad-col/enliecaalhkhhihcmnbjfmmjkljlcinl).

(The Globe and Mail, previously a partner on the ad collector, has [taken over stewardship](https://www.theglobeandmail.com/politics/article-globe-and-mail-takes-over-global-facebook-ad-monitoring-project/) of the project which was previously maintained by ProPublica.)

We're asking our readers to use this extension when they are browsing Facebook. While they are on Facebook a background script runs to collect ads they see. Optionally, if the user wants to help train the political classifier, they can vote on whether or not a particular ad is political. Server-side, we use those ratings to train a naive Bayes classifier that then automatically rates the other ads we've collected. The extension also asks the server for the most recent ads that the classifier thinks are political so that users can see political ads they haven't seen. We're careful to protect our user's privacy by not sending identifying information to our backend server.

We're open sourcing this project because we'd love your help. Collecting these ads is challenging, and the more eyes on the problem the better.


### Download and try it

* Download the [Facebook Political Ad Collector for Chrome](https://chrome.google.com/webstore/detail/facebook-political-ad-col/enliecaalhkhhihcmnbjfmmjkljlcinl?hl=en)
* Download the [Facebook Political Ad Collector for Firefox](https://addons.mozilla.org/en-US/firefox/addon/facebook-ad-collector/)


### Run it on your own

The collector is broken out into a series of services, each with its own repo:

- **[`fbpac-extension`](https://github.com/globeandmail/fbpac-extension)**: The browser extension that monitors your Facebook feed and sends ads data to the app.
- **[`fbpac-api`](https://github.com/globeandmail/fbpac-api)**: Rails app. Serves API for React portion of website (public-facing, if applicable, and admin); manages login for admin dashboard; serves entire Targeting Breakdown page. Runs on Amazon ECS.
- **[`fbpac-backend`](https://github.com/globeandmail/fbpac-backend)**: Rust app. Receives ads and ad ratings submitted by extension users; serves a rudimentary feed for Ads Others Are Seeing tab in extension; serves assets for website. Runs on Amazon ECS.
- **[`fbpac-classifier`](https://github.com/globeandmail/fbpac-classifier)**: Python scripts for building a model to train a model to predict whether an ad is political given its text, along with scripts for updating each database record with that prediction. Runs on Amazon ECS.
- **[`fbpac-archiver`](https://github.com/globeandmail/fbpac-archiver)**: A cron to archive the database to a CSV file weekly for analysis, if necessary. Runs on Amazon ECS.
- **[`fbpac-page`](https://github.com/globeandmail/fbpac-page)**: A simple site showing how to install the ad collector and explaining WTF the FBPAC project is.

Instructions on how to install the different parts of the project are contained in the individual repos, in their respective READMEs. Most of the repos rely on Docker for deployment, so make sure you have that installed.

To learn more about how to internationalize the app for a new country, read the [i18n guide](https://github.com/globeandmail/facebook-political-ads/tree/master/INTERNATIONALIZATION.md).

The app is designed to be deployed to an Amazon Web Services Elastic Container Service (ECS) environment.

For the ads database, we use an Amazon RDS db.t2.medium Postgres database, but that may be beefier than necessary. However, to my knowledge, it hasn't gotten overloaded.

We also need an Amazon S3 folder for storing ad images; the Rust app has to have credentials to write to this bucket.


### Stories

* [Globe and Mail takes over global Facebook ad-monitoring project](https://www.theglobeandmail.com/politics/article-globe-and-mail-takes-over-global-facebook-ad-monitoring-project/)
* [Nine early signs of how Facebook ads are being used in Ontario’s election](https://www.theglobeandmail.com/canada/article-ford-targeting-the-trump-curious-nine-things-weve-learned-so-far/)
* [Help Us Monitor Political Ads Online](https://www.propublica.org/article/help-us-monitor-political-ads-online)
* [Mehr Transparenz im Schatten-Wahlkampf](http://faktenfinder.tagesschau.de/wahlkampf-facebook-dark-ads-101.html)
* [Bringen Sie Licht in den dunklen Facebook-Wahlkampf](http://www.sueddeutsche.de/digital/bundestagswahl-bringen-sie-licht-in-den-dunklen-facebook-wahlkampf-1.3656582)
* [Wie werben die Parteien auf Facebook?](http://www.spiegel.de/netzwelt/games/facebook-political-ad-collector-plugin-sammelt-wahlwerbung-auf-facebook-ein-a-1166566.html)
* [So werben die Parteien auf Facebook](http://www.spiegel.de/netzwelt/web/facebook-political-ad-collector-parteienwerbung-auf-facebook-im-ueberblick-a-1169154.html)
* [Warum Zuckerberg den deutschen Wahlkampf durchleuchten ließ ](http://www.sueddeutsche.de/digital/werbung-auf-facebook-und-google-warum-zuckerberg-den-deutschen-wahlkampf-durchleuchten-liess-1.3679603)
* [Trust Issues](https://www.wnyc.org/story/on-the-media-2017-09-22/)
* [Facebook Allowed Questionable Ads in German Election Despite Warnings](https://www.propublica.org/article/facebook-allowed-questionable-ads-in-german-election-despite-warnings)
* [Versuch der anonymen Einflussnahme auf den Bundestagswahlkampf](http://www.sueddeutsche.de/digital/facebook-versuch-der-anonymen-einflussnahme-auf-den-bundestagswahlkampf-1.3713694)
* [Bundestagswahl: Versuch anonymer Einflussnahme](https://www.ndr.de/fernsehen/sendungen/zapp/Greenwatch-Versuch-anonymer-Einflussnahme,greenwatch100.html)
* [Same-sex marriage survey: help us track targeted ads on Facebook ](https://www.theguardian.com/australia-news/2017/oct/17/same-sex-marriage-survey-help-track-targeted-ads-facebook)
* [Adani posts weird video ad on Facebook to fend off Carmichael criticism](https://www.theguardian.com/business/2017/oct/21/adani-posts-weird-video-ad-on-facebook-to-fend-off-carmichael-criticism)
* [How Malcolm Turnbull, GetUp and Adani are using Facebook ads to push their agenda](https://www.theguardian.com/technology/2017/oct/25/how-malcolm-turnbull-getup-and-adani-are-using-facebook-ads-to-push-their-agenda)
* [Hjælp os med at kortlægge politiske reklamer på Facebook](https://www.information.dk/indland/2017/11/hjaelp-kortlaegge-politiske-reklamer-paa-facebook)
* [Facebook Allowed Political Ads That Were Actually Scams and Malware](https://www.propublica.org/article/facebook-political-ads-malware-scams-misleading)
* [Helfen Sie uns, verdeckte Polit-Werbung zu enttarnen](https://www.republik.ch/updates/polit-werbung-enttarnen)


### Types of ads the collector doesn't collect

 - mobile ads
 - pre-roll, midstream video ads
 - video from ads in the stream
 - Instagram-only ads (Note that many ads are set to run on Facebook and Instagram with the same creative)


### Where we need your help (aka TODOs)

In general, the project needs more tests. We've written a couple of tests for parsing the Facebook timeline in the extension directory, and a few for the tricky bits in the server, but any help here would be great!

Also, the rust backend needs a bit of love and care, and there is a bit of a mess in `https://github.com/globeandmail/fbpac-backend/tree/master/server/src/server.rs` that could use cleaning up.

We could also use help in orchestrating the deployment of all the repos to AWS. Right now, it's still a bit of a manual process to get the ECS tasks set up, permissions, whatnot. It'd be great to have something like a [Terraform](https://www.terraform.io) script that would set everything up in one fell swoop and make redeployment or environment changes a breeze.

A few other TODOs:

 - considering triggering the ad parsing routine only on scroll, to mitigate the clicking-off problems.
 - consider retaining utm params (i.e. a whitelisted set of parameters in links that are added by advertisers, not by FB and ipso facto are not personally identifiable, e.g. `utm_content`, etc., since those sometimes include useful metadata about the ad.)
 - consider turning off the panelist_ads table, etc.
 - consider seeding the partisanship model in new languages with political tweets.
 - the [targeting parser](https://github.com/globeandmail/fbpac-backend/tree/master/server/src/targeting_parser.rs) is in need of internationalization
