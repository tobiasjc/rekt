# REKT - a collections' type setup for Apache Nutch

This setup project is meant to offer an easier deployment for the different configurations you might have to setup when crawling specific websites with unique configurations for each. The purpose of making this as an automatic process is for the case of a cluster deployment using tools such as `Docker` and/or `kubernetes` - in such environments you should try to stay away from the lower level configuration and bring automatic tools to your operations. Those configuration files are not meant to change the crawler functionalities in any way, just to configure them easier.

## Why?

The first reason is as already stated: making it easier to setup clusters that might crawl multiple websites that needs a very specialized and unique configuration that might not be shared accross multiple domains. The second reason is to provide others a way to crawl what is really needed from such websites through a curated configuration, respecting the information that might be stored into them while extracting the information that is needed for our studies or tools.

Such websites should really have well structured `robots.txt` specifications that people should be happy to follow - but this is not what happens 99% of the time. Also, some of those websites can be really tricky to deal with, for example with api calls or wordpress strcutures.

## The configuration

From what I've seen in my use cases, the whole configuration thing boils down to six pieces of the Apache Nutch plugable system:

- The nutch script, which is the lower level of interaction with the multiple phases of the Apache Nutch crawling process - `bin/nutch`
- The crawl script, which is the higher level of abstraction over the nutch script - `bin/crawl`
- A list of urls to be used as seeds - when crawling a single website this seed might be unique - `urls/seeds.txt`
- A regex configuration that might let you filter the pages of this website down to your purpose - `conf/regex-urlfilter.txt`
- A configuration specific to the crawler behaviour when managing this website content - `conf/nutch-site.xml`
- A way to get the content so it can go through your processing phases setting up a message broker or other data storage technologies - `conf/index-writers.xml`

And so this is all you'll get inside every single of those collections. I advise you reading the configurations and tweaking them for your best interest. For example, all the configurations include the indexing of the raw 'binary data' crawled and base64 encoded - because I think this would apply to most people doing this kind of website crawling, but it might not apply to you. The indexing from all the collections is also a bit modified and only includes the `Kafka Indexer` for some standard configurations - again, you might like to change it.

## Running the setup

The setup is very easy to use. The program semantics are:

```bash
setup.sh [-n | --nutch-home] NUTCH_HOME [-c | --collection] COLLECTION_NAME
```

Where the `[-n | --nutch-home] NUTCH_HOME` is simply the home of your nutch instalation - the home is the directory where your bin and conf directories reside. The `[-c | --collection] COLLECTION_NAME` name is one of the available inside the `collections` folder.

## The magic

The script is as simple as it gets, it's only a copy of the configurations of any collections to your `NUTCH_HOME` path, and nothing more. The only important action of this script is in single line, that is:

```bash
cp -rvf "${COLLECTIONS_FOLDER}/${COLLECTION}"/* "${NUTCH_HOME}"
```

ps: remember that this `${COLLECTIONS_FOLDER}` variable is by default the `REKT_HOME` variable concatenated with a directory called `collections`. The `REKT_HOME` can be altered via script parameter so you have a better modularized way to run the script from different paths.

## Example

So for example, if your nutch installation path is `/opt/apache/crawler/nutch` and you want to setup the crawler for the `dlmf` collection, you can run the `rekt` script as:

```bash
./rekt -n /opt/apache/crawler/nutch -c dlmf
```

Or, for example, if you have the `rekt` installed on `/usr/local/bin` path and you have your collections at your `/root/` directory, you can use the `rekt` script as:

```bash
rekt -r /root/ -n /opt/apache/crawler/nutch -c dlmf
```

The script will now understand the `REKT_HOME` variable as being the `/root/` and utilize the `collections` directory from it to setup the installation at `NUTCH_HOME` as set before.

After that you can start your Apache Nutch crawling process with a usual basic command like:

```bash
${NUTCH_HOME}/bin/crawl -i -s ${NUTCH_HOME}/urls/ ${NUTCH_HOME}/crawl -1 
```

Just as you normally would.
