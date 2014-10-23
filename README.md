# Insights Log Reader
## A Log Reader for New Relic Insights

### Requirements

- A New Relic Insights account. Sign up for a free account [here](http://newrelic.com)
- node.js: http://nodejs.org/download/
- Network access to New Relic (proxies are supported, see details below)

### Installation & Usage Overview

1. Download the latest version of the agent: https://github.com/sschwartzman/Insights-Log-Reader/archive/master.zip
2. Unzip on the server server that you want to monitor
3. Configure `config/config.json` 
  * Start by copying `config.json.template` OR your pre-built `config.json` to `config/config.json`.
  * [Click here for `config.json` configuration details](#configjson)
4. Start agent using either the node command-line, or a wrapper package like [forever](https://github.com/nodejitsu/forever)
  * `nohup node /path/to/app.js > /path/to/InsightsLogReader.log 2>&1 &`
  * Directions for usage with [forever](https://github.com/nodejitsu/forever) coming soon...
5. Login to New Relic Insights UI and check the Data Explorer for events flowing in.
  * In Insights, click the "Explore" Icon from the top of the left-hand menu. 
  * In the Data Explorer, look for "LogEvent" as an event type, and there to be a non-zero count of events for that type.
  * Click on "LogEvent" to see some of the most recent events that match your search criteria.
6. Make dashboards! Here are helpful links to New Relic's docs:
  * [Building Dashboards](https://docs.newrelic.com/docs/insights/new-relic-insights/managing-dashboards-and-data/building-insights-dashboards)
  * [Editing Dashboards](https://docs.newrelic.com/docs/insights/new-relic-insights/managing-dashboards-and-data/editing-insights-dashboards)
  * [Using Widgets](https://docs.newrelic.com/docs/insights/new-relic-insights/managing-dashboards-and-data/using-widgets)

### <a name="configjson"></a> Configuring the `config.json` file

#### Required settings

* `accountid` - Your account ID, which can be found in any New Relic RPM or Insights URL in the format: http://insights.newrelic.com/{account_id}/...
* `apikey` - Your API Insert Key, which is created and saved here: https://insights.newrelic.com/accounts/{account_id}/manage/api_keys
* `logfile` - Path and name of the log file you wish to tail.
* `parsers` - An array of parsers, each of which will be run against the log file. 

#### Parser settings 

* `name` - (Optional) Name of the matching parse will appear in debug logs.
* `comment` - (Optional) Anything you want can go here. Currently, I'm using it to store an example log line for this parser.
* `match` - Regular Expression used to search your log file. Read below in [Regex-ing Your Log](#regex) for details.
* `headers` - Column names for matched fields in a line of your log file.
  * You MUST have as many header fields as you do match fields. For now.
* `eventtype` (Optional) The type of event you want to put into Insights - can be anything. Defaults to 'LogEvent'

##### `timestamp` in `headers`:

  * `timestamp` is a special header in Insights that can be populated from your data.
  * If you use `timestamp`, it must be parsing a standard date & time format. For example: `2014/07/01 21:23:09.841 GMT`
  * If the timestamp that is read is more than 24 hours old, it is ignored (Insights limitation).
  * The original contents of the timestamp field are saved as `timestamp_orig` in each Insights record.
  
#### Optional settings

* `newmsgs` - If set to true, Insights Log Reader will only look for new log events. If false, Insights Log Reader will read the whole log every time it is started.
* `send_interval` - how often (seconds) it will send events. Default is 10.
* `send_max_events` - max # of events it will send in an interval. Default is 50.
* `send_min_events` - min # of events necessary in an interval to send. Default is 10.

### <a name="regex"></a> Regex-ing your log

* Standard regex with groupings is used.
* Each grouping `(...)` will produce a field in the log event.
* `\` must be escaped with another `\`, like in the example below.
* Here is an example regex used to find most any log event in an Mac OSX system.log (/var/log/system.log):
  `(\\w{3}[0-9\\s -]+)(\\S+)\\s([^\\[]+)\\[(\\d+)\\] -(.*)`,
* More regex examples to follow!
* I recommend using a regex tool like [regexr](http://www.regexr.com/) or [reggy](http://reggyapp.com/) to build your regex statement.
    * Copy-and-paste in a sample line or lines from the log file in question to build the regex.
