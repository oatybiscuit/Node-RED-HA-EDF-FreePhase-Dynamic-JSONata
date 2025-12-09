# Node-RED-HA-EDF-FreePhase-Dynamic-JSONata

[![GitHub Release](https://img.shields.io/github/v/release/oatybiscuit/Node-RED-HA-EDF-FreePhase-Dynamic-JSONata)](https://github.com/oatybiscuit/Node-RED-HA-EDF-FreePhase-Dynamic-JSONata/releases)
![GitHub Downloads (all assets, latest release)](https://img.shields.io/github/downloads/oatybiscuit/node-red-ha-edf-freephase-dynamic-jsonata/latest/total)
[![License: MIT](https://img.shields.io/github/license/oatybiscuit/node-red-ha-edf-freephase-dynamic-jsonata)](/LICENSE)

[![Node RED](https://img.shields.io/badge/Node--RED-8f0000)](https://nodered.org/)
[![JSONata](https://img.shields.io/badge/JSONata-285154)](https://jsonata.org/)
[![Home Assistant](https://img.shields.io/badge/Home_Assistant-038fc7)](https://www.home-assistant.io/)

A **Node-RED** flow to read, process, and store into context variables, **EDF FreePhase Dynamic Tariff Prices** using **JSONata**. Ability to create sensor in **Home Assistant** for prices and bands, with potential to use and display these in graph and table format.

![node-red-flow](/images/NodeRED_flow.png)

>[!NOTE]
> I don't use EDF or Octopus Agile myself, so this was formed out of personal interest rather than necessity. It does work, but it is a *learning example* rather than active home automation forged in the white heat of real need.

## Introduction

This flow is a cut-down copy of my Octopus Agile flow. Dealing only with EDF FreePhase dynamic as input price tariff, and only to obtain the prices and provide summary information on the band prices.

## What this Node-RED flow does

<details>
<summary> Flow action </summary>

This flow will trigger once every day to:

- Read EDF FreePhase dynamic tariffs for the most recent 48 hours (96 records)
  - add local time (DST aware) to the provided UTC timestamps
  - save this tariff array to flow context
  - save the array update outcome (debug) details to flow context
  - provide an HA sensor with the current and next prices, updated every half hour
  - provide also the full tariff array in a sensor attribute
  - provide weekly updated product and tariff information to flow context

</details>

## Prerequisites

> [!WARNING]
> If you are not comfortable running Node-RED then please do not attempt to use this code! This code uses JSONata throughout rather than function nodes and JavaScript. JSONata is a declarative-functional language. This may require more time and effort to understand, modify and debug the code *should you wish to make changes*.

<details>
<summary> Installation requirements </summary>

This is a **Node-RED** flow, written to run in Node-RED alongside Home Assistant. You will need:

- **Node-RED**, ideally running as a [Home Assistant add-on](https://github.com/hassio-addons/addon-node-red#readme)
- The EDF Product *references* for your own or chosen EDF dynamic *product*, *tariff*, and pricing *area*
- The [WebSocket nodes](https://github.com/zachowj/node-red-contrib-home-assistant-websocket) installed and their HA server configuration working correctly

</details>

## EDF Dynamic Tariffs

<details>
<summary> EDF and API calls </summary>

**EDF** provide API calls to access tariff information. These product calls are open and do not need any authorization. The calls do require a valid product reference, and the correct area code. Kraken is the backbone system used, so these details are almost identical to the Octopus Agile API arrangements.

You can find out more about [Octopus Energy API Documentation - Agile](https://developer.octopus.energy/docs/api/#agile-octopus) in their documentation.

The API call used is of the form:

`https://api.edfgb-kraken.energy/v1/products/EDF_FREEPHASE_DYNAMIC_12M_HH/electricity-tariffs/E-1R-EDF_FREEPHASE_DYNAMIC_12M_HH-L/standard-unit-rates/?page_size=96`

The key elements in this are the EDF *product* name **EDF_FREEPHASE_DYNAMIC_12M_HH** and the *tariff* name **E-1R-EDF_FREEPHASE_DYNAMIC_12M_HH**. Other EDF tariff products may be available. The **L** is for *my* region (South-West) here in the UK. The [regions are explained in detail here](https://www.energy-stats.uk/dno-region-codes-explained/)! You will need to identify your own **product** and **tariff**, or select a current EDF Dynamic offering. Note that the product you use will need to be updated if your account product changes. *If you have access to your account-tariff details via other integrations, then you may be able to automate a look-up so as to provide notification should the product used here no longer aligns with your account product.*

Every day, usually reliably around 12:00 local time but certainly before 22:00, EDF publish the next set of records by adding to the end of the tariff file. The records are for each 30 minute period, starting at the hour and the half-hour. Thus the file grows by 48 records every day. Tariffs are published using UTC time only. They are priced on the European market, so issued from CET midnight, and therefore run from 23:00 today to 23:00 tomorrow UK time. For the sake of my sanity, each daily set will be called 'today' and 'tomorrow', with the 'tomorrow' set including the two records 23:00-23:30 and 23:30-00:00 from today.

> The flow maintains only the most recently published 96 records - hence this should always be the full set of 'yesterday' and 'today' up to the tariff update at around 12:00, and the full set for 'today' and 'tomorrow' after the update at around 12:00.

### Daily and half-hourly updates

Every half hour (on the hour and at 30 minutes past) the current tariff price will be fetched from the context tariff array and presented in the Home Assistant sensor. The flow will count the *remaining* records in the array at each update, and when less than 20 will initiate new API calls. Where tariff updates are delayed, this means that the flow will continue to poll the API regularly until a successful read occurs.

</details>

## Installing the flow

As is standard, the Node-RED flow is contained within a JSON file. The file contents can be copied, and imported using the usual Node-RED import from clipboard facility, or loaded directly from file.

> [!TIP]
> If you are updating from an earlier version:- Disable *all* the inject trigger nodes, *all* the flow entity sensor nodes *and* their corresponding sensor configuration nodes *and redeploy* first! This 'shuts down' the old flow (which you can then keep as a backup) and reduces the risk of potential duplication of existing entity sensors! You can also manually delete all the context variables as these will be recreated afresh with the new flow. You may well see warning messages that you are importing nodes that already exist!

<details>
<summary> Installing the Node-RED code </summary>

> To avoid potential issues with an update, disable the inject, sensor and sensor-configuration nodes and redeploy, so as to first remove the existing entity registrations in Home Assistant. You can delete the existing flow entirely before importing the update, however as long as the Home Assistant sensor nodes are fully disabled the existing flow can usually remain without issue (you may just see a warning that the nodes you are importing already exist).

- Save the Node-RED flow from here (see the release file). In Node-RED go to the hamburger menu, select ‘import’, and import the JSON flow file.

- Add any missing nodes to your palette.

- Deploy and test that the API calls work by manually triggering the Inject node and checking that the context is updated with saved variables.

</details>

### Setting your parameters

**The API calls in the flow use product parameters which need setting to *your* particular product, tariff, and region.**

<details>
<summary> Product and region setting </summary>

The parameters are stored in the **Tariff** node, inside a JSON object. The **mode** field can be 'import' only. For import, the **product** and **tariff** names are required. Usually the tariff name will be "E-1R-" *product* "-X" where X is your specific region.

```javascript
{
    "region": "L",
    "mode": "import",
    "import": {
        "product": "EDF_FREEPHASE_DYNAMIC_12M_HH",
        "tariff": "E-1R-EDF_FREEPHASE_DYNAMIC_12M_HH-L"
    },
    "export": {
        "product": "",
        "tariff": ""
    }
}
```

The export tariff prices will be ignored since EDF do not currently provide Dynamic Export products, and the `mode` should always be left as `import`.

If you are using EDF, then you can obtain your product code and region from your current account or bill.

If, like me, you are using this for interest only, then you need to search the EDF products for something that suits, and check your region using the link above. The flow will also populate a product context variable, which contains the latest EDF dynamic product offerings, so one of these can be used.

Make the necessary changes, manually delete the *edfProducts* context variable (to force this to be updated) and test that the flow works by using the Inject node, and looking at the flow context variables. A working flow will produce an *edfAgileTariff* variable in flow context containing a 96-record array, and product details for your chosen tariff.

</details>

### Connection to Home Assistant

As good practice, the Home Assistant WebSocket node has been [scrubbed](https://zachowj.github.io/node-red-contrib-home-assistant-websocket/scrubber/) of their Home Assistant service configuration details, and *additionally disabled*. **You will need to edit these nodes, *and* the node configuration node behind each one, *and possibly the Home Assistant server configuration node itself.***

> [!IMPORTANT]  
> As a *minimum*, you will need to edit the sensor configuration nodes, at least to update and redeploy, in order to reconnect these configuration nodes with *your* default homeassistant server.

<details>
<summary> Connecting to Home Assistant </summary>

- If you are not already running a working connection with Home Assistant, make sure that you have installed the WebSocket nodes in the Node-RED palette, the [Node-RED companion integration](https://github.com/zachowj/hass-node-red) in Home Assistant, and you have correctly configured the *Home Assistant server configuration node*.
  - You should already be able to see a global context variable in Node-RED called `homeassistant` which contains a copy of the Home Assistant state.

In full detail:

- double click each Home Assistant Sensor node in turn to edit
  - enable the node (option bottom left)
  - click the edit pencil next to the *Entity config* entry to edit the ha-entity-config node
  - enable the ha-entity-config node (option bottom left)
  - check the *Server* entry - this should be HomeAssistant or similar
    - if you *do not* have a working HomeAssistant server, use the *Add new server...* option and set up the server
    - if you do have a working HomeAssistant server, ensure this is correctly selected in the *Server* entry field
  - update the *server* node, the *ha-entity-config* node, and finally save the *sensor* node and redeploy the flow once you have updated all the sensor nodes

![configuration-setup](/images/configsetup.png)

</details>

### Other settings

> [!CAUTION]
> The tariff array is passed to Home Assistant as a sensor attribute. The Home Assistant Recorder has a capacity limit and will generate an error message in the Home Assistant logs, to the effect that this entity state and attributes are too large and are not being saved by the Recorder.

There is no way to overcome this without reducing the size of the array. The error messages *are advisory* but can be prevented by adding the following to your Home Assistant configuration file. This prevents the Recorder from attempting to save this particular sensor.

```yaml
recorder:
  exclude:
    entities:
      - sensor.edf_freephase_prices
```

## Using this Node-RED flow

The flow should run without issue. EDF tariff price updates are expected to be very reliable. Since the EDF FreePhase offering is a simple three-band tariff, price updates are happening around midday every day. The flow has error checking but no recovery - if the API calls fail then they will be repeated hourly until they succeed. Error checking will look to see that the API call has returned 96 records, covering a full 48 hours. The array will not be updated until the return is correct, complete, and extends into tomorrow.

If the EDF Tariff data is corrupt or fails to update after 23:00, then there is not much we can do about it.

### Daylight Saving Time (DST)

<details>
<summary> How DST and local time is managed </summary>

EDF provide all dynamic tariffs with times based on UTC. For the UK this is the same as GMT in the winter, but local time becomes BST or UTC +1 in the summer. Ideally device switching should operate using UTC throughout so as to avoid DST change issues, however to facilitate 'local time' for display and control, the flow creates a DST change-over record in context, and uses this to add local time to each record.

Daylight Saving Time (DST) is calculated using the European / UK rule of 01:00 UTC on the last Sunday in March and in October. This information is held in context, refreshed for each new year, and updated on every API update.

> DST changes and using 'local time' rather than UTC can give rise to timing issues. Whilst this flow provides UTC, local time, and DST change information, and has been successfully tested over BST to GMT transition, care should be taken when using local time rather than UTC, and when dealing with periods that span over DST time changes.

</details>

### Context variables

<details>
<summary> Details of all the flow context variables </summary>

#### edfDST

Holds sufficient information to be able to determine the current timezone setting and the next change, with fields for:

- the current year
- last update timestamp and Unix ms
- DST start and stop, as timestamps and Unix ms
- current DST offset as ms
- midpoint between DST on and off
- current timezone as "GMT" or "BST"
- DST on, DST changing shortly, and direction of change Boolean flags
- timezone changing from and to ("GMT"/"BST")

This is created on first update, recreated each new year, and updated on every API call.

#### edfAgileTariff

Holds an *array* of **96 periods**, each period being an *object* with fields for:

- period from and to timestamps in UTC
- price (inc VAT)
- separate fields for date and times
- local date and times
- DST changing flag
- ISO format from and to timestamps
- set ["old" for today, "new" for tomorrow]
- band: R, A, G for red, amber and green
- position [start, middle or end period in each band]

This array is updated on successful API update, and 'tomorrow' will *in effect* replace 'yesterday'.

#### edfUpdate

Holds an *object* with details of the most recent API update call, with fields for:

- the unix timestamp of the most recent update attempted (whether successful or not)
- import, and export success flags ['--' not requested, 'ER' array error, 'MT' returned but not yet updated for tomorrow, 'OK' - all correct]
- match, will be false *if* both import and export requested *and* their periods do not match (true otherwise)
- success, which will be true only if the one or both requests are correct and both update into tomorrow
- the array from and up to timestamps (span), and separate import and export array end timestamps
- hours left in the stored array, hours available in new update (remaining maximum time for either import/export) and hours left in new update (minimum of import/export array remaining)
- product checking for import and export; will show "not the latest product" if the product in use is not included in the latest Agile Product list

Note that, since edfUpdate is constructed and saved *before* the edfProducts information is updated, edfUpdate will have incomplete or incorrect product checking details after the very first call, or whenever the called product is modified.

#### edfProducts

Holds details created at the first API call (if edfProducts missing in context) and then updated once each week (default Sunday). Calls the product and tariff API in EDF to populate details on the import product being used, as well as an array of the latest dynamic products available.

The import product given as the API calling parameter will show with the respective product:

- code
- name
- term (months)
- starting date and ending date (end date will be null if no end date set)
- sale flag: "no-end" if ending date is null, "sunset" if ending date set, "closed" if the ending date is now in the past
- brand
- available regions list
- full region tariff code used
- standing charge and unit rate for this product and region (at the *exact* time of the update, so of little practical use for Agile products)

The *latest product list* provides all currently market-available Octopus variable-rate products that have 'Agile' in the name:

- mode (import/export)
- product code
- product term in months
- brand
- start offering date
- end offering date (will be null if there is no current end date)
- an http link to the full product details

#### edfBands

Holds an object with arrays for all pricing band start/ends over the 48 hours, blocks as band periods, and prices with the G,A,R band prices for today and tomorrow.

For blocks:

- sequence count
- price
- band "G", "A", or "R"
- from and upto timestamps in both local and UTC formats
- periods as count of half-hourly periods in the block
- duration in minutes

For prices:

- band "G", "A", "R" in increasing price order
- old price (today/yesterday) and new price (tomorrow/today)
- change, as ratio of new price to old price as percentage

</details>

### Home Assistant Sensors

#### EDF Prices

<details>
<summary> EDF FreePhase Dynamic Prices </summary>

The *state value* is the UTC timestamp string for the current half-hour period start time.

Attribute fields hold:

- array: a reduced (fields removed) copy of the 96 entry tariff array
- values for price for the current half-hour period, and the next half-hour period
- a count of the number of forward periods remaining in the tariff array
- the current band, and the next band [RAG] for the half-hour period
- the next new band period [band RAG change]
- the start and end time of the next new band period
- the new band period price
- array: of all bands as start/end records
- array: of all blocks [combined start/end]
- array: prices for each band [GAR in price order]

This sensor is updated every half-hour.

Some fields in the array object have been deleted before passing to the sensor. This is to reduce the overall size of the attribute array. The deleted fields can be modified by adding or removing field names from the transformation-delete operation in the 'Value Now' Change node if required.

</details>

## Display

### Sensor entities

The provided variables are mostly held in the sensor *attributes*, and these can be extracted within Home Assistant using appropriate template functions and configuration settings. Attributes are used for passing the arrays, out of necessity, however the flow could easily be modified to add further entity sensors, should you prefer to pass attributes as single entities.

As an example, the main Agile tariff **sensor.edf_freephase_prices** can provide a display of the current and next prices. The card shown here is a standard entities card, with the YAML configuration as given below.

![agile entities](/images/EDF_values.png)

<details>
<summary> Entity card YAML configuration </summary>

```yaml
type: entities
entities:
  - entity: sensor.edf_freephase_prices
    name: Time period
    secondary_info: last-updated
    icon: mdi:av-timer
  - type: attribute
    entity: sensor.edf_freephase_prices
    attribute: periods_left
    name: Periods Left in array
  - type: attribute
    entity: sensor.edf_freephase_prices
    attribute: price_now
    name: Price - now
    suffix: p
    icon: mdi:currency-gbp
  - type: attribute
    entity: sensor.edf_freephase_prices
    attribute: band_now
    name: Band - this period
    icon: mdi:book
  - type: attribute
    entity: sensor.edf_freephase_prices
    attribute: price_next
    name: Price - next period
    suffix: p
    icon: mdi:currency-gbp
  - type: attribute
    entity: sensor.edf_freephase_prices
    attribute: band_next
    name: Band - next
    icon: mdi:book
  - type: attribute
    entity: sensor.edf_freephase_prices
    attribute: new_band_next
    name: Next Band
    icon: mdi:arrow-right
  - type: attribute
    entity: sensor.edf_freephase_prices
    attribute: new_band_start
    name: Next Band starts at
    icon: mdi:clock-alert-outline
  - type: attribute
    entity: sensor.edf_freephase_prices
    attribute: new_band_end
    name: Next Band ends at
    icon: mdi:clock-alert-outline
  - type: attribute
    entity: sensor.edf_freephase_prices
    attribute: new_band_price
    suffix: p
    name: Next Band Price
    icon: mdi:currency-gbp
title: EDF Freephase

```

</details>

### Tables

The sensor provides several *arrays* of values, and one of the better ways to display array data is using tables. I use the custom component [flex-table-card](https://github.com/custom-cards/flex-table-card), which can display a sensor attribute array as a table.

#### FreePhase Bands

<details>
<summary> FreePhase pricing bands in a table display </summary>

The FreePhase Bands array is a good example of what can be shown. This table has been coded to show the timestamp in *local time* (so BST in summer) and to show only the active and remaining table going forward from the current period.

![agile tariff as a table](/images/EDF_price_table.png)

The custom flex-table-card requires a configuration to achieve this, provided as shown below. Note the use of a hidden column and the 'strict' setting in order to select only the current and future periods.

```yaml
type: custom:flex-table-card
strict: true
title: EDF FreePhase Price Bands
entities:
  include: sensor.edf_freephase_prices
columns:
  - data: blocks
    modify: x.band
    name: Band
  - data: blocks
    modify: >-
      (new Date(x.from)).toLocaleString("en-GB", {day:"2-digit", month:"short",
      hour:"2-digit", minute:"2-digit", timeZoneName:"short"})
    name: Start Time
  - data: blocks
    modify: |-
      let disp=(new Date(x.upto) > new Date());
      if (disp) 'yes';
    name: TEST
    hidden: true
  - data: blocks
    name: Period
    modify: |-
      const p=(new Date(x.upto) - new Date(x.from))/60000;
      var hours = ('00' + Math.floor(p/60)).slice(-2)
      var minutes = ('00' + (p % 60)).slice(-2);
      `${hours} h ${minutes}`
  - data: blocks
    modify: x.price
    name: Price (p)
```

</details>

### Graphs

I use the custom [apexcharts-card](https://github.com/RomRider/apexcharts-card) to display the tariff prices. This requires several graph configuration settings as well as data generator code to get the display I want. The graph shows the EDF FreePhase dynamic price, over a complete 48 hour period of 'today' and 'tomorrow', as well as annotations for peak (Red) and off-peak (Green) periods, and the current offered standard variable-tariff prices for my region.

![agile tariff graph](/images/EDF_graph.png)

<details>
<summary> EDF FreePhase graph configuration </summary>
Card settings:

Graph_span is set to 48h, and span to start at the beginning of the day offset by -1h so the graph runs from 23:00 yesterday to 23:00 tomorrow.

Tariff series as step-line, using a data generator. This takes the *attribute.array* and pulls the *from* (timestamp) and *price* mapping from the full array to the new time/price array required for the chart. The step-line will plot horizontally from the start (from) of the period data-point to the next data point. At the end of the line, one extra data point is needed to make the last entry run horizontally on to the end of the day. Hence the last `final.push` adding the upto timestamp (of the last period) and the last period price.

Additional annotations are added to cover the EDF FreePhase Band periods of 23:00 (day before) - 06:00 for 'Green' and 16:00 - 19:00 for 'Red'. Horizontal annotations add the fixed-tariff prices current for comparison. The graph 'Now' option does not work with annotations (it stops them showing) so extra code is added to self-generate a 'now' line for completeness.

Configuration code is given below:

```yaml
type: custom:apexcharts-card
header:
  show: true
  title: EDF Freephase Prices
show:
  last_updated: true
now:
  show: false
  label: now
graph_span: 48h
span:
  start: day
  offset: "-1h"
series:
  - entity: sensor.edf_freephase_prices
    data_generator: |
      let prices = entity.attributes.array;
      let ends = prices.length-1;
      let final = prices.map((item, index) => {
        return [item.from, item.price];
      });
      final.push([prices[ends].upto, prices[ends].price]);
      return final;
    curve: stepline
    name: Import
    show:
      legend_value: false
      extremas: true
    stroke_width: 2
    color: red
apex_config:
  annotations:
    yaxis:
      - "y": 23.16
        borderColor: red
        label:
          text: Ref
          borderWidth: 0
          style:
            background: "#0000"
    xaxis:
      - x: EVAL:new Date().setHours(16,0,0)
        x2: EVAL:new Date().setHours(19,0,0)
        fillColor: "#FEB019"
        label:
          text: Peak
          borderWidth: 0
          style:
            background: "#0000"
      - x: EVAL:new Date(Date.now()-86400000).setHours(23,0,0)
        x2: EVAL:new Date().setHours(6,0,0)
        fillColor: "#B3F7CA"
        label:
          text: Cheap
          borderWidth: 0
          style:
            background: "#0000"
      - x: EVAL:new Date(Date.now()+86400000).setHours(16,0,0)
        x2: EVAL:new Date(Date.now()+86400000).setHours(19,0,0)
        fillColor: "#FEB019"
        label:
          text: Peak
          borderWidth: 0
          style:
            background: "#0000"
      - x: EVAL:new Date(Date.now()).setHours(23,0,0)
        x2: EVAL:new Date(Date.now()+86400000).setHours(6,0,0)
        fillColor: "#B3F7CA"
        label:
          text: Cheap
          borderWidth: 0
          style:
            background: "#0000"
      - x: EVAL:Math.floor(Date.now()/1800000)*1800000
        label:
          text: Now
          position: bottom
          borderWidth: 1
yaxis:
  - min: 0
```

</details>

## JSONata

Is a very different language, so if you are looking to either understand the code used here, or to use and modify, then you are probably going to have to learn a bit about JSONata.

The JSONata documentation can be found at [JSONata documentation](https://docs.jsonata.org/overview.html)

There is a great [sandbox](https://try.jsonata.org/) which I use for all my development work.
