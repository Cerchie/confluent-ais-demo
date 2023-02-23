# ais-demo

## Overview

Follow along this tutorial-style demo to learn how to set up [Confluent Cloud](https://confluent.cloud) and analyze data using [ksqlDB](https://ksqldb.io/). We'll use AIS, which is an [automatic tracking system](https://en.wikipedia.org/wiki/Automatic_identification_system) used by ships, and pull live public data from it on ships' speed, location, and other details. We'll then feed that data in to an Apache Kafka® topic via a connection to Confluent Cloud. Afterwards, we'll build streams using ksqlDB and run analyses on that data. 

## Pre-requisites

1 - Sign up for a [Confluent Cloud](https://confluent.cloud) account. 

2 - Make sure you have `jq` (which is a utility that formats JSON results), `gpsd` (which formats AIS data) and `kcat` (which lets you see what's going on in a Kafka cluster) by running these commands in your terminal:

`jq --version` 

`gpsd --version`

`kcat -V` 


2a - If you need to install them: 

View the [instructions](https://stedolan.github.io/jq/download/) for installing `jq` depending on your operating system. 

For `gpsd`, `brew install gpsd` for MacOS,

or 

`apt-get install gpsd-clients` for Linux.

unfortunately, installation is [not recommended for WSL2](https://gpsd.gitlab.io/gpsd/installation.html) and linux distribution is recommended. 

Install kcat using the [instructions](https://github.com/edenhill/kcat) from the repo README. 

## Setting up Confluent Cloud

Sign in to your Confluent Cloud account. Head over to the [confluent.cloud/environments](https://confluent.cloud/environments) page and click 'Add Cloud Environment' on the top right of your screen. 

<img width="1459" alt="click Add Cloud Environment on the top right" src="https://user-images.githubusercontent.com/54046179/220384774-b7518172-d674-4f92-ab80-6b4ac7aa6cf4.png">

Name your environment 'ais_data' and click 'Create'. Note: If you're prompted to select a Stream Governance package, just click the 'I'll do it later' link at the bottom of the page. 

On your cluster page, click 'Create cluster on my own' or 'Create cluster'. 

<img width="1361" alt="click 'Create cluster on my own'" src="https://user-images.githubusercontent.com/54046179/220385552-680095bc-e927-4148-bcf7-7f94bb1790ff.png">

Select the basic configuration. 

<img width="1705" alt="Screen Shot 2023-02-21 at 8 21 07 AM" src="https://user-images.githubusercontent.com/54046179/220385813-5024e575-cacd-468f-9416-e23befebcc7d.png">

Then, select your prodiver and region. Next, name your cluster 'ais_data', and click 'Launch cluster'. You'll be re-directed to your cluster dashboard.

Use the left-hand navbar to navigate to the API key page, create an API key (give it global scope) and download the values for later. 

<img width="1086" alt="Use the left-hand navbar to navigate to the API key page" src="https://user-images.githubusercontent.com/54046179/220386731-f178c915-12c7-41c1-9c47-cdfc1c1ff163.png">

Now, navigate to 'Topics' in the left-hand navbar, and create a topic named 'ais' using the default values. 

<img width="1606" alt="navigate to 'Topics' in the left-hand navbar" src="https://user-images.githubusercontent.com/54046179/220387497-837bf9ff-c2c3-428e-9868-1d7f2201c23c.png">

That's all for now. We'll revisit the Confluent Cloud environment in a minute so keep that tab open! 

## Connecting to the websocket

To connect to the website and view formatted JSON results, run this command in your terminal:

```
nc 153.44.253.27 5631|gpsdecode |jq --unbuffered '.'
```

Note: you can ctrl+C or click the terminal to pause the flow of data. 

You'll see results similar to:

```
{
  "class": "AIS",
  "device": "stdin",
  "type": 1,
  "repeat": 0,
  "mmsi": 257017920,
  "scaled": true,
  "status": 0,
  "status_text": "Under way using engine",
  "turn": 0,
  "speed": 0,
  "accuracy": false,
  "lon": 6.080573,
  "lat": 61.863708,
  "course": 300.2,
  "heading": 115,
  "second": 28,
  "maneuver": 0,
  "raim": false,
  "radio": 81923
}
```

These results might seem a little magical if you're not familiar with `nc` and `jq`, so let's break it down. 

`nc` is a [netcat](https://linuxize.com/post/netcat-nc-command-with-examples/) command that reads and writes data across network connections. Here, we're connecting to the [ais websocket](https://www.kystverket.no/en/navigation-and-monitoring/ais/access-to-ais-data/) located at IP `153.44.253.27` and port `5631`.

The `gpsdecode` command after the first pipe does what it sounds like: decodes the gps data. 

Lastly, passing the `--unbuffered` flag to `jq` flushes the output after each JSON object is printed. As far as the `'.'` goes, you can create objects and arrays using `jq` syntax (see [examples](https://developer.zendesk.com/documentation/integration-services/developer-guide/jq-cheat-sheet/)) and `'.'` is a way of saying "Put this all in a top-level JSON object, if you please jq". 

## Feeding the websocket results to Confluent Cloud

Now, run this command to pipe in the data from the source to your topic in Confluent Cloud using kcat. Where the command says `YOUR_API_KEY_HERE` and `YOUR_API_SECRET_HERE`, replace those values with the api key and secret you downloaded earlier. 

To find the `YOUR_BOOTSTRAP_SERVER_HERE` value, click on the 'Cluster Settings' tab on the left-hand navbar, and look under 'Endpoints'. You'll see your value there. It's also in the credentials file you downloaded earlier. 

```
nc 153.44.253.27 5631 |
gpsdecode |
kcat -X security.protocol=SASL_SSL -X sasl.mechanism=PLAIN -b YOUR_BOOTSTRAP_SERVER_HERE -X sasl.username=YOUR_API_KEY_HERE -X sasl.password=YOUR_API_SECRET_HERE -t ais -P
```

## Confirming input in Confluent Cloud

Navigate to your `ais` topic in the Confluent Cloud interface using the left-hand navbar, click the 'messages' tab, navigate to a partition using the dropdown offset menu, and view the messages coming in! 

<img width="1714" alt="Navigate to your `ais` topic in the Confluent Cloud interface using the left-hand navbar" src="https://user-images.githubusercontent.com/54046179/220414720-b35c6e03-aada-491a-961b-710e6bccb8b7.png">

# TENTATIVE NEXT STEPS

## Setting up ksqlDB

## Analyzing data with ksqlDB

