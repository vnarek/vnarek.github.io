---
title: Project wordoc
draft: "true"
topic: project-idea
---
# Description and MVP

Thesaurus on steroids available as an browser extension. It highlights infrequent word on hover gives you more information about the word like:
* synonyms
* antonyms
* etymology
* word tree from different languages
* ...
For starters only english language will be supported. We need a storage for the words and interesting information about them. This can be probably done by taking the data from the [wiktionary dump](https://dumps.wikimedia.org/enwiktionary/20240401/) in combination with online thesaurus etc.

# Tech-stack

I could write this in a combination of GO on the backend and JS on the frontend, which would be probably easier for me to actually finish. I would not learn anything new. Maybe would in regards to the JS, but not on the backend.

Or I can go more unconventional route and treat this as an toy/hobby project for me to try new stuff. I could write the API in .NET with F#. This would make the finishing of the project much harder.

An interesting compromise could be to implement this fast in GO and then try to rewrite the minimal viable product in F# with .NET or another language / ecosystem.

For the database I would probably go with some document database to store the information about the words and in-memory hash-table for now to easily retrieve the word frequencies or other factors on which I want to base my filtering. After the filtering we can ask for the information about the words in the document database.