# max480-random-stuff website

This is (part of) the source code for the max480-random-stuff.appspot.com website, a [Google App Engine](https://cloud.google.com/appengine/) app.

It contains the full source for:
- [the Everest Update Checker frontend service](https://max480-random-stuff.appspot.com/celeste/everest_update.yaml)
- [the Celeste custom entity catalog](https://max480-random-stuff.appspot.com/celeste/custom-entity-catalog)
- [the everest.yaml validator](https://max480-random-stuff.appspot.com/celeste/everest-yaml-validator)
- [the Everest Update Checker status page](https://max480-random-stuff.appspot.com/celeste/update-checker-status)
- [the help page for the Mod Structure Verifier bot](https://max480-random-stuff.appspot.com/celeste/mod-structure-verifier?collabName=CollabName&collabMapName=CollabMapName&assets&xmls&nomap&multiplemaps&badmappath&badenglish&misplacedyaml&noyaml&yamlinvalid&missingassets&missingentities)
- Some [GameBanana](https://gamebanana.com)-related APIs for the Celeste community described in the few next sections.

If you want to check how the update checker's everest_update.yaml file is generated, check [the Everest Update Checker Server repo](https://github.com/max4805/EverestUpdateCheckerServer) instead.

## The GameBanana search API

This API uses the mod search database generated by [the Everest Update Checker server](https://github.com/max4805/EverestUpdateCheckerServer) to find mods based on keywords. **This searches Celeste mods only**.

It is used by [Olympus](https://github.com/EverestAPI/Olympus), the Everest installer and mod manager, to search Celeste mods on GameBanana.

To use this API, call `https://max480-random-stuff.appspot.com/celeste/gamebanana-search?q=[search]`. The answer is in yaml format, and is a list of the top 20 matches. For example:

```
$ curl https://max480-random-stuff.appspot.com/celeste/gamebanana-search?q=spring+collab+2020
- {itemtype: Map, itemid: 211745}
- {itemtype: Gamefile, itemid: 13452}
- {itemtype: Map, itemid: 212317}
- {itemtype: Gamefile, itemid: 13185}
- {itemtype: Gui, itemid: 35325}
- {itemtype: Gamefile, itemid: 13258}
- {itemtype: Map, itemid: 212999}
- {itemtype: Gamefile, itemid: 12784}
- {itemtype: Gamefile, itemid: 9486}
```

Here the top 2 results are https://gamebanana.com/maps/211745 (The 2020 Celeste Spring Community Collab) and https://gamebanana.com/gamefiles/13452 (2020 Spring Collab Randomizer).

The search engine is powered by [Apache Lucene](https://lucene.apache.org/). It supports, among other things:
- searching for a **phrase**: `"Spring Collab"`
- **OR** and **NOT** keywords: `Spring Collab NOT Randomizer`
- searching for other fields in GameBanana submissions:
  - the **name** (default): `name: (Spring Collab 2020)`
  - the GameBanana **type**: `type: gamefile`, `type: map`, etc.
  - the GameBanana **ID**: `id: 9486` (those can be found at the end of links to mods)
  - the GameBanana **category**: `category: Textures`
  - one of the **authors**: `author: max480`
  - the **summary** (line that appears first on the page): `summary: (grab bag)`
  - the **description**: `description: "flag touch switches"`

For a full list of supported syntax, check [the Lucene documentation](https://lucene.apache.org/core/8_7_0/queryparser/org/apache/lucene/queryparser/classic/package-summary.html#package.description).

## The GameBanana sorted list API

This API allows to get a sorted list of most downloaded, liked or viewed Celeste mods on GameBanana.

If you want to retrieve the latest mod with no type filter, it is recommended to use [the real GameBanana API](https://api.gamebanana.com/docs/endpoints/Core/List/New) instead, for more up-to-date information.

The URL is `https://max480-random-stuff.appspot.com/celeste/gamebanana-list?sort=[sort]&type=[type]&category=[category]&page=[page]` where:
- `sort` is the info to sort on (**mandatory**). It can be `latest`, `likes`, `views` or `downloads`
- `type` (or `itemtype`) is the GameBanana type to filter on (optional and case-insensitive). For example `Map`, `Gamefile` or `Tool`
- `category` is the GameBanana mod category ID to filter on (optional), this is returned by [the GameBanana categories list API](#gamebanana-categories-list-api). For example `944`
- `page` is the page to get, first page being 1 (optional, default is 1). Each page contains 20 elements.

The output format is the same as the GameBanana search API, [see the previous section](#the-gamebanana-search-api).

## GameBanana categories list API

This API allows getting a list of GameBanana item types _that have at least one Celeste mod in it_ (contrary to [the official GameBanana API for this](https://api.gamebanana.com/Core/Item/Data/AllowedItemTypes?&help)), along with how many mods there are for each category.

The counts returned by this API might not match the numbers displayed on the GameBanana website; that's because GameBanana counts mods that do not show up in the list.

Each entry has either an `itemtype` or a `categoryid` (for mods that have `itemtype` = `Mod`, so that they can be filtered by category instead). The "All" entry is special and has an empty `itemtype`, and carries the total number of mods.

The URL is `https://max480-random-stuff.appspot.com/celeste/gamebanana-categories?version=2` and the result looks like:
```yaml
- itemtype: ''
  formatted: All
  count: 541
- itemtype: Effect
  formatted: Effects
  count: 2
- itemtype: Gamefile
  formatted: Game files
  count: 114
- itemtype: Gui
  formatted: GUIs
  count: 11
- categoryid: 944
  formatted: Textures
  count: 2
...
```

Not passing `?version=2` will result in only `itemtype`s getting returned, with one of them being `Mod`. This is here for backwards compatibility.

## GameBanana WebP to PNG API

This API simply downloads a WebP image from GameBanana, and converts it to PNG. **This can only be used with GameBanana screenshots.**

Usage example: https://max480-random-stuff.appspot.com/celeste/webp-to-png?src=https://screenshots.gamebanana.com/img/ss/gamefiles/5b05ac2b4b6da.webp

This API may answer with a 302 (redirect) leading to the actual image on Google Cloud Storage. Images are cached there for 30 days, to avoid having to convert it each time.

## GameBanana file order API

This API just gives the IDs of the files present on a few GameBanana submissions. You can get multiple mods at a time (up to 20), like this:
`https://max480-random-stuff.appspot.com/celeste/gamebanana-fileorders?itemtypes=Mod,Map&itemids=53749,211745`

The response will look like this, returning the file info on all requested mods in the same order as they were passed:
```yaml
- ['532232', '531562', '530584', '527697']
- ['545472', '484937', '539975']
```

With the example request, this means `['532232', '531562', '530584', '527697']` is the file list for Mod 53749, and `['545472', '484937', '539975']` is the one for Map 211745.

If you request a mod that does not exist, you will get an empty array as a result.

This was done since the GameBanana API gives this information through the order of the entries in the `Files().aFiles()` dictionary. Order shouldn't matter in JSON dictionaries though, so libraries often mess it up.