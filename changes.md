# Changelog

## Version 1.0.0

- First stable release

## Version 1.0.1

### Fix incorrect rain mapping

Storing the actual floating-point value of the rain amount for calculation would cause the rain amount to be reported too low due to floating-point precision errors.

Instead, only the count of spoon trips (an integer) is stored.

### Additional rain observations

Additionally, the rain mapping now exposes three more values:
 - spoon trips since last record
 - rain rate in spoon trips per hour
 - size of spoon in inches

The first uses the unit group `group_count`, the second one `group_rate` (see below) and the latter `group_rain`.

### New unit group `group_rate`

A new unit group `group_rate` is defined. Currently it contains only one unit `per_hour`.
`group_rate ` is intended for observations measuring the instantaneous rate of some event. I.e. how often something it would happen in 60 minutes if the event continued to occur at the current rate. This is comparable to "rain rate" except that it is intended for concrete, countable events.

The driver only uses it for the rate of the rain collector spoon tripping.

### Other bugfixes and changes

- Improve error handling in UDP broadcast receiver
- Improve closing of UDP broadcast receiver
- Make rain mapping and wind service `None`-safe
- Convert all caught errors in driver to `InitializationError ` and `WeeWxIOError `. This improves integration with WeeWX's automatic retries.
- Minor refactoring

## Version 1.0.2

- **Fix incorrect rain diff calculation**
  Previous amount needs to be subtracted from the current amount, not
  vice-versa.

## Version 1.0.3

- **Fix broadcast activation failing after some time**
  After an irregular timespan (usually hours to days) broadcast activation would silently fail. This was due to the polling and broadcast activation happening at the same time. The WLL however is unable to cope with multiple simultaneous HTTP requests.
  This was resolved by implementing a centralized scheduler to avoid multiple tasks being executed simultaneously.

- **Switch all driver threads to "daemon mode"**

## Version 1.0.4

- **Fix high CPU usage** ([#2](https://github.com/michael-slx/weewx-weatherlink-live/issues/2))
  Due to not blocking the loop when no packets were available, this driver caused WeeWX to have an absurdly high CPU utilization. This is now resolved by pausing the loop until a packet is received (or an error is caught). As a safety measure, the driver checks for new data every 5 seconds anyway.

## Version 1.0.5

- **Fix interrupts and signals being caught in main loop** ([#4](https://github.com/michael-slx/weewx-weatherlink-live/issues/4))
  Interrupts and signals were caught in main loop, preventing WeeWX from being stopped using `Ctrl + C` or other signals used by the service manager.
- **Fix log messages in the main packet loop not respecting `log_success` configuration option** ([#3](https://github.com/michael-slx/weewx-weatherlink-live/issues/3))
- **Completely rework scheduling logic**
  All scheduling is now centralized. This should prevent any tasks to be executed immediately one after another. Push refreshing is now performed every 20 minutes and is scheduled using a integer factor of polling events

## Version 1.0.6

### General

- **Add data packet watchdog** ([#11](https://github.com/michael-slx/weewx-weatherlink-live/issues/11))

  If no data packet is emitted for a configurable count of iterations, an error will be raised, in turn crashing the engine (and potentially triggering a restart).
  The count of iterations can be configured by adding the `max_no_data_iterations` option in the driver's section (default is 5). An iteration will happen every 5 seconds when no data is received.

  Thanks to user [cube1us](https://github.com/cube1us) debugging this issue.

### HTTP

- **Retry failed HTTP requests**
  If HTTP requests fail, the driver will attempt to send them again up to 3 times. Should none of the attempts succeed, an exception will be raised.

- **Make HTTP requests honor `socket_timeout` option** in configuration ([#11](https://github.com/michael-slx/weewx-weatherlink-live/issues/11))

  Thanks to user [cube1us](https://github.com/cube1us) debugging this issue.

### Configuration

- **Configure accumulators for custom observations defined by driver**
  `rainCount` is accumulated by summation and for `rainSize` the last available value is used.
- **Move example configuration from installer tool to configuration editor**
  Example configuration now includes comments.
- **Remove `polling_interval` from default stanza**
  It isn't a required option as a sensible default is set.

### Logging

- **Change log message to not read like an error**
  Mapping debug message previously read like an error message. This is now improved.
- **Reduce chattiness of driver during normal operation** ([#10](https://github.com/michael-slx/weewx-weatherlink-live/issues/10))
  The driver previously logged `INFO` messages when emitting packets. This was changed to `DEBUG`.
- **Add additional log messages when receiving and decoding broadcast packets**
  When receiving broadcast messages, the size of the packet will be logged and a message will be printed when creating packet objects.

## Version 1.0.7

- **Fix packet emission loop**

  With the changes of the previous release, push (broadcast) packets prevailed to the point where no push packets were emitted anymore.

## Version 1.0.8

- **Add `appTemp` option for THW and THSW index mappers**

  THW and THSW mappers can have an `appTemp` option. This will map the values to the `appTemp`/`appTemp1` field additionally to the respective custom fields.

- **Add battery status flag mapper (`battery`)**

  Mapper `battery` maps battery status indicator flags of transmitters to `batteryStatus` fields with respective transmitter ids (i.e. `batteryStatus1` to `batteryStatus8`).

- **Fix error message when an unknown mapping type is used in configuration**

- **Fix broken temperature-only mapping**

## Version 1.0.9

- **Allow named mapping targets for battery status**

  `battery` mapping now maps one transmitter only, but supports mapping to WeeWX's standard named battery status fields additionally to the numeric ones.

- **Fix mapping targets being used multiple times**

## Version 1.0.10

- **Fix `THW`/`THSW` mappers occupying an `appTemp` field even if not marked with `appTemp`**
- **Fix `THW`/`THSW` mappers not being able to resolve map targets if `appTemp` is already used**

## Version 1.0.11

- Fix mapping targets of leaf temperature and wetness being specified as sets ([#15](https://github.com/michael-slx/weewx-weatherlink-live/issues/15))
- Fix wrong formatting syntax for packet keys

## Version 1.1

- **Add command to print mappings**

Use the shell command `wee_device --print-mapping` to display a table of mappings. The command will print a table grouped by transmitters and mappings.

- **Add alias for driver entry point and schema definition**

An alias for the driver entry point and the database schema definition was added so that the driver is recognized by the setup utility `wee_config`.

For backwards compatibility, the old entry point can still be used.

- **Add interactive setup utility**

Using the command `wee_config` to run an interactive setup will now bring up prompts specifically for this driver. The hostname as well as mappings can be configured this way.

- **Add driver-specific logging settings**

Logging of successful and erroneous operations can now be separately configured for the driver using the `log_success` and `log_failure` options respectively. Driver-specific options take precedence over global options.

- **Tweak the levels of some log messages**

In light of the change listed above, the level of some log messages was tweaked. Most notably, the message that a new packet was received was raised to the `INFO` level. You can disable these by setting `log_success` to `false` either globally or just for the driver.

- **Configuration value boundary checks**

Configuration values are checked to be within bounds immediately during initialization.

- **New documentation**

Documentation was expanded and split into multiple separate documents.

## Version 1.1.1

- **Fix linting, spelling and compatibility issues**

Driver is now compatible with **Python 3.7 or later**.

- **State required Python version in documentation**

## Version 1.1.2

- **Fix installation issue** on WeeWX 5 beta

## Version 1.1.3

- **Update documentation for WeeWX 5**

- **Add an [Upgrade guide](docs/upgrading.md)**

- **Deprecate driver alias module**

  The driver alias module `user.weatherlink_live_driver` is no longer necessary in WeeWX 5 and will be removed in a future release. See [Upgrade guide](docs/upgrading.md) for more information.

- **Simplify the configuration prompts**

  Some of text was removed to make the process less intimidating ... Also the mapping template prompt now always appears.

- **The extension installer will now add an example config.**

  The snippet contains sensible defaults for a WeatherLink Live with a standard Vantage Pro II Plus.

## Version 1.1.4

- **Remove `[Station]` section from installer example config** ([PR #39](https://github.com/michael-slx/weewx-weatherlink-live/pull/39))

  Specifying this section in the example config caused it to be removed when the extension was uninstalled, breaking the whole configuration file.

  Thanks to user [tkeffer](https://github.com/tkeffer) for submitting the pull request.

- **Fix type-hinting issue and compatibility problems**

  WeeWX supports Python 3.7 but some type-hinting used by this driver required at least Python 3.9. This was fixed so that the driver now works with Python 3.7.

## Version 1.1.5

- **Fix resource leaks and cleanup issues** ([#42](https://github.com/michael-slx/weewx-weatherlink-live/issues/42))

  WeeWX does not close and re-instantiate the driver module when an error
  occurs. This previously led to issues when attempting to re-open the UDP socket.

  Additionally, there was a crash when cancelling scheduler tasks.

- **Make retry log messages more intuitive** ([#47](https://github.com/michael-slx/weewx-weatherlink-live/issues/47))

  Previously, retries of HTTP requests were counted starting at `0`. This 
  was changed to start at `1` (= first retry = second attempt).

- **Add log messages for successful request retries** ([#46](https://github.com/michael-slx/weewx-weatherlink-live/issues/46))

  When a HTTP request succeeds, a respective message is logged.
  If it took more than one attempt, a message is logged with
  the `error` level.

- **Set `max` accumulator for `rainRate` metric** ([#44](https://github.com/michael-slx/weewx-weatherlink-live/issues/44))

  Rain rate is accumulated by using maximum value for consistency
  with WeatherLink app/dashboard.
